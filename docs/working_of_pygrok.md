# Woking of pygrok.py


### Introduction

    Pygrok is a python library to parse strings and extract information from structured/unstructured data
### Sample code :

    1. from pygrok import Grok
    2. text = 'gary is male, 25 years old and weighs 68.5 kilograms'
    3. pattern = '%{WORD:name} is %{WORD:gender}, %{NUMBER:age} years old and weighs %{NUMBER:weight} kilograms'
    4. grok = Grok(pattern)
    5. print(grok.match(text))
       #Output : {'name': 'gary', 'gender': 'male', 'age': '25', 'weight': '68.5'}

### Explation :

    1) When Grok(pattern) is called, we have a  DEFAULT_PATTERNS_DIRS(path of predefined patterns which contains multiple files which contains a pattern name and the regular expression.
       Example : USERNAME [a-zA-Z0-9._-]+) the constructor is called.

    2)  The constructor does the following things :

        (a) 'pattern'(argument) is stored for the current object ('custom_patterns_dir'=None, 'custom_patterns'={} are the other arguments set by default).
            prefdefined_patterns is a dictionary which stores pattern name (as defined in the patterns file) and a Pattern object(which has pattern name and regex as attributes).
            prefdefined_patterns calls '_reload_patterns(DEFAULT_PATTERNS_DIRS)'.

        (b) In '_reload_patterns(DEFAULT_PATTERNS_DIRS)' we take the path where multiple files are located.
            On each file we call '_load_patterns_from_file(os.path.join(dir, f))''.
            We return the dictionary with pattern name and Pattern object.

        (c) '_load_patterns_from_file(os.path.join(dir, f))' takes each file, strips off the extra spaces in the end and beginning using strip() function and ignores the comments using 'startswith(‘#’)'.
            With each line we find a space and split the line into two variables 'pat_name = l[:sep]'(before space, pattern name) and 'regex_str = l[sep:].strip()'(after space, regular expression).
            (Example : pat_name = USERNAME, regex_str =[a-zA-Z0-9._-]+) and create Pattern object and save it in the dictionary. This function returns this dictionary.

        (d) Main work in the constructor is in a while loop which starts with :

          i.  Storing the type of data, of the variable given as input in pattern, in 'type_mapper'.
              Example(%{WORD:name:int} ; name->int).
              'type_mapper' is the dictionary where is we store the variable name and type of it.

          ii. Most important work is done in this step using the
                          're.sub(pattern, repl, string, count=0, flags=0)'
              of regex library where we replace the pattern with the regular expression using the pattern dictionary we stored in above steps.

              Here 'pattern' argument has the value (r'%{(\w+):(\w+)(?::\w+)?}')
                   'repl'    has the value          (lambda m: "(?P<" + m.group(2) + ">" + self.predefined_patterns[m.group(1)].regex_str + ")")
                   'string'  has the value          (py_regex_pattern(pattern given as input))

              lambda fucntion here replaces the found py_regex_pattern, as described by first argument repl, by looking the name of pattern in predefined_patterns.


    		  Example :

      		Before :
              %{WORD:name} is %{WORD:gender}, %{NUMBER:age} years old and weighs %{NUMBER:weight} kilograms
      		After :
              (?P<name>\b\w+\b) is (?P<gender>\b\w+\b), (?P<age>(?:%{BASE10NUM})) years old and weighs (?P<weight>(?:%{BASE10NUM})) kilograms

          iii. After this we compile the pattern above using compile in regex library.


    3) When grok.match(text) is called, which does the following :

      (a) We call search(text) (given as input) with the pattern (as specified above) and store it in match_obj.

      (b) Here is the second most important step.
          Using groupdict() of regular expression library which return a dictionary containing all the named subgroups of the match, keyed by the subgroup name.

      Example :

      Before :
      match_obj =   <regex.Match object; span=(0, 52), match='gary is male, 25 years old and weighs 68.5 kilograms'>
      After :
      {'name': 'gary', 'gender': 'male', 'age': '25', 'weight': '68.5'}
