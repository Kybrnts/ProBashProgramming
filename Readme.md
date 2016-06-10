# ProBashProgramming
Experimenting with a "C-Style" programming approach to Chris Johnson's "Pro Bash Programming" Scripting in the GNU/Linux
Shell, 2nd edition.

Introduction
------------
Bash is a powerful and very flexible scripting tool, that has a lot of potential for fast-developing programs/utilities,
specially for Linux Sysops and Sysadmins. Yet in my experience code developed in bash is oftentimes difficult to
read/understand and hard to maintain and/or extend. Its hard-to-learn syntax is prone to uncanny and sometimes buggy
expressions.
Over the past 10 years or so I used Shell scripting as one of my main resources to develop anything from the most simple
threshold-triggered alerts sent via email, to log cleaners, back-up automated tasks, to more complex ones, like software
installers or service load balancers; some of which loaded configuration values from a separate conf. file, or generated
reports in HTML format. In any case, success rate ranged from "thrilling achievement" to "disastrous fail", although 
overall results improved with time. This was for the most part, thanks to a knowledge input that came from internet
forums, blogs, advice from colleagues and a handful of e-books that I could gather over time, such as Machtlet Garrels's
"Bash-Beginners guide" and Mendel Cooper's "Bash-Advanced guide". Among those it was Chris Johnson's 
"Pro Bash Programming", notorious for its procedural/modular programming approach.

C-Style Approach
----------------
In spite of my very limited programming knowledge, I remembered from my student days, most of the issues regarding BAD
programming practices, that I was starting to notice in my own scripts; and some of the benefits of using proper C's
structured and modular approach. This approach is not enforced by any shell, however with enough time, one can find a 
lot of features in Bash that provide support for it. I remember by starting to use an idea once found in a web forum: to
wrap all script's code inside a function declaration. This allowed among other things to end execution's flow and set an
exit status using the "return" built-in instead of using the exit call. Naming such a function "main" was the natural
next step. One small drawback is that we still need to call this main function at the end of the script for the 
execution to take place. Moreover script's positional parameters are to be passed to it using "$@" or "$*" if needed.
But although Bash's functions support helped with the procedural approach, the variable scope constraints imposed by the
"local" built-in allows for better modularity. In addition, it is easy to create functions capable of writing values to
variables in the calling environment by simulating a passed-by-reference argument mechanism, as it will be shown (As of
bash-4.1 or later, this is natively supported with "declare -n").
Finally we are also allowed to create non-executable files containing only bash declaration statements, and later 
include that code in any bash script by using the "source" built-in. This allows code reusability through library files.

Goals
-----
The idea is to establish a set of "good bash scripting rules", and then put them to the test by re-writing all the code
available in Chris Johnson's book following them. The task shouldn't be very hard as published code is already close to
that.  However it is expected that this rules will be improved and extended during the process. 
From a more practical point of view we also aim to improve:
* Code clarity and simplicity, making it easy to maintain, share and extend.
* Code Reusability, putting a bit of effort in function libraries development.
* Also some effort in writing code that it is safer to execute, robust, that works consistently with wider range of inputs
  and with proper run time error handling. 

Scripting rules
---------------
Below rules will be observed in order to meet above goals. They also explain why code within this small project looks
the way it does, and also why, so often, I needed to write two versions of the same program (providing same
functionality): One as it appears in the book, and then another complaint w/this rules. This is particularly notorious
when declaring functions that write its output to referenced names, instead of writing it to a global variable.

General coding rules:
* Performance
  * Prefer built-ins over external commands;
  * But, use fast external commands to process very large inputs;
  * Avoid sub-shells and pipelines.
* Quotation
  * When referencing a variable, enclose its name in double quotes (double quoting) to prevent:
    * Reinterpretation of all special characters within the quoted string;
    * Word splitting;
  * When referencing a variable in the RHS of an ASSIGNMENT, double quoting is not needed (in Bash);
  * Use double quoting for arguments to present them as single words, even when containing white spaces.
* Arithmetic
  * Use names "integer" attribute, for integer variables (local -i aname=0);
  * Perform arithmetic operations and numeric comparisons within "(( ))" blocks: e.g. if ((42<=24+24)), ((3**3==27));
  * Reference variables by name, not expansion, within arithmetic evaluation: e.g. ((i++)) rather than (($i++)),
    ((v+=42)) rather than v=$(($v+42)).
* Safety and reliability
  * Avoid using uppercase-only names for variables (to avoid name collisions with bash environment's);
  * Use underscore-prefixed uppercase names for global variables;
  * Declare and intitialize variables before using them;
  * Avoid using the "eval" built-in with parameters, to avoid the risk of code injection;
  * In particular "printf -v $ref_NAME [FORMATSTR] STRVALUE" is preferred over "eval "$ref_NAME=\"$intstr\"" to write
    a value into referenced name. It also allows control over STRVALUE's format (no performance drawbacks detected);
  * Do not start local variable names with underscores unless it is meant to be written by an user-defined function;
  * Use "_<funcname>" for variable names that are to be written by functions (passed-by-reference variables); where
    "funcname" is function's name itself or a shorthand version of it. This convention reduces the chances of name
    collisions, when passing variable NAMES to functions so that they can write values to those names, and hence
    received names could potentially be already defined locally;
  * When changing directory, confirm that it worked afterwards;
  * When issuing an external command, confirm that it worked afterwards.
* Clarity
  * Prefer lines of 120 characters of length or less;
  * Use functions to improve readability and control scope;
  * Document each function with at least two fields: DESCRIPTION and USAGE.
* Portability
  * Avoid using the "declare" built-in. Use local instead (other shells do not support it);
  * Prefer "[" to "[[" for conditional expressions;
  * Try to keep code as POSSIX complaint as possible, but do not sacrifice Safety/reliability.

Global declarations rules:
* Put all source (includes) at the beginning of the script.
* Then declare and initialize all new global variables.
* Then declare all functions.
* If the script is to be executed (not sourced), declare a main function.
* Main() must be the last one to be declared so that it can use all above declared functions.
* Call "main "$@"" as the last script command to execute it passing all arguments to it.
* Avoid flow control outside main(): This means that ONLY below commands are allowed OUTSIDE main's declaration:
  * Source files (or exit if it fails), to include external declarations;
  * Function and (global) variable declarations.

Function declaration rules:
* Create a function header w/local variables as arguments to store all positional parameters values. This allows
  using custom and more meaningful names for arguments and so ease variable tracking throughout function's body;
* Declare MANDATORY (non-unset) non-null arguments first, then MANDATORY that can be null and finally the optional ones
  (may be unset or null);
* Prefix parameters that hold variable names (references) with "ref_", and make them MANDATORY and non-null;
* Use the following parameter expansions right before assignment to arguments:
  * "Display error if null or unset", for MANDATORY non-null positional parameters;
  * "Display if null", for MANDATORY positional parameters that may be null strings; 
  * "Use default value" or "Use alternate value" parameter expansion for OPTIONAL positional parameters;
  This will enforce proper function call by stopping script execution if a MANDATORY argument is missing, or null when
  it is not supposed to. Also assign default values for optional parameters;
* Then flush all parameters with "shift $#", to "enclose" function's header and to release memory; also any remaining
  parameters passed to the function by mistake will be ignored;
* Then declare as "local" ANY needed variables inside the function (avoid globals);
* Use return to avoid exit calls; however they might be used upon signal traps or die()-like functions.
