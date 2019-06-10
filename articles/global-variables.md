# Global Variables

Global variables are essential to advanced SAS programming. However, they are
disguised as "macro variables". Now there _is_ support for scoping macro
variables, but in practice I found this rarely used.

## The Problem

Global variables seem pretty innocent for cases like the following:

```
%let var=mydataset;
%let tmp=a_value;

data &var;
  ...
run;

proc print &var title="&tmp";
```

And indeed it is. This `.sas` file is limited in scope. Further, it allows you
to very quickly see _how_ the code is parameterized. You may even consider
"packaging" this code up as a macro for reuse in other programs.

```
%macro mymacro(var)
tmp=a_value;

data &var;
  ...
run;

proc print &var title="&tmp";

%end;
```

But what if we use this macro elsewhere:

```
%let tmp=test;

%mymacro(var = "blah")
```

Oops! I used the macro variable `tmp` in two places. We now have a "namespace
collision." That is: a variable defined in two different scopes with
(potentially) two different values. 

The _solution_ here is to use `%local` within my macro definition to declare
that the `tmp` variable is for local use only.

However, I propose that the use of macro variables as global variables is
a bad pattern overall. The simple example I shared above is _much_ more sanitized
than what I experienced in industry, and often I saw things more like this:

```
%macro myrealworldmacro(dataset=dataset)
  %put &somevar;

  data mylib.&dataset;
    %if &othervar = "by_client";
      by &clientvar;
    %endif;
    
    %for i=1 %to 30;
      %do &&client&i;
    %end;
  run;

%end
```

To summarize the problems above:
 - `%local` is not used anywhere
 - what is `&somevar` / `&othervar` / `&clientvar`?
     - inputs declared / defined by the calling process
     - important inputs that this macro will not run correctly without
 - why are they not defined as formal inputs?
     - this macro is never called outside of a particular context
     - that context always has these variables defined
     - my time is valuable, and specifying inputs is annoying
     - oh, it is also hard / pointless to document macro variable inputs (have to do it in comments)
 - why must `&dataset` live in `mylib`?
     - because I don't want a `&lib` parameter
     - because all of our datasets are in `mylib`

Many of these answers are natural for a data scientist's day-to-day work. After
all, they are not software developers and will not be evaluated on principles
such as DRY and code maintainability. In fact, some data scientists will not
have a code review at all, are slammed with work / deadlines, developed this in
production, and will deploy it to automation without any review.

However, the long term problems that will come with this code are:
 - hard to reason about and reuse
 - dangerous to call in unknown contexts / with other macros using similar variables
 - very brittle / fragile design

## A better way

I propose that the _real_ problem here is: if macros are a hammer, they are the
biggest, most powerful programming hammer in a SAS programmers toolbox. Further,
there are few other programming tools so big and powerful in the SAS language.

As a result, SAS programmers (often very smart individuals) just develop conventions
and patterns for using these hammers in ingenious ways, while remembering all of the
ways to _not_ shoot themselves in the foot.

When moving into the R programming language, all I wanted was a macro. I
lamented, "Why is there no macro language for R!?" I felt helpless and time
constrained: like everything was way too much work without my hammer.

I eventually learned to embrace _functional programming_ with all its benefits. Further,
I learned that functional programming is an elegant solution to the macro problem. It 
presents better solutions without the drawbacks. I do have to define input variables,
but it makes my life much easier, and I can reason about things more easily now. Further,
I can declare input (and OUTPUT) types other than "string", which is very important.

In functional programming, I found a sledge hammer, power tools, saws, and screws.
