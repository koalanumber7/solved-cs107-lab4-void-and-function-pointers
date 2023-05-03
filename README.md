Download Link: https://assignmentchef.com/product/solved-cs107-lab4-void-and-function-pointers
<br>
During this lab, you will:

explore how C void*/function pointers support generic functions practice writing callback functions

implement a generic function that operates on data of any type

First things rst! Find an open computer to share with a partner. Introduce yourself and tell them about your plans for Halloween.

<h1>Get started</h1>

Clone the repo by using the command below to create a lab4 directory containing the project les.

<h1>Lab exercises</h1>

<h2>1) Code study</h2>

<h3>memmove</h3>

The C library provides a handful of raw memory routines (e.g. memcpy , memset , …) that operate on data of unspeci ed type. Let’s take a look inside memmove (version below from musl (http://www.musl-libc.org)) to better understand how these kind of functions are implemented.

Go over the code with your partner and discuss these questions:

<ol>

 <li>The function’s interface declares its parameters as void* pointers, but internally it</li>

</ol>

manipulates these pointers as char* . Why the inconsistency? What would be the consequence of trying to reconcile the discrepancy by declaring the interface as char* or changing the implementation to use void* ?

<ol>

 <li>Note that there is no typecast on lines 3 and 4 when assigning from an untyped pointer to</li>

</ol>

a typed pointer. A void* is the universal donor/recipient and can be freely exchanged with other pointer types, no cast necessary. I nd the author’s decision to not cast pleasing, given my preference for casting only where you absolutely must. As I mentioned in lecture, the internets argue about this endlessly

(https://stackover ow.com/questions/605845/do-i-cast-the-result-of-malloc) if you want to hear more about why it is contentious.

<ol>

 <li>What special case is being handled on line 6?</li>

 <li>What special case is being handled on line 7? What is the di erence between the functions</li>

</ol>

memcpy and memmove ? (Check the man page for information)

<ol>

 <li>What two cases are being divided by the if/else on Lines 9/12? Why are both cases</li>

</ol>

necessary?

<ol>

 <li>Despite lines 10-11 and lines 13-15 both accomplishing a similar function (i.e. copy bytes from source to destination), they are oddly dissimilar in construction: for instead of</li>

</ol>

while , pointer arithmetic/dereference versus array indexing. Which version do you nd

easier to follow? Which one is easier for you to verify correctness? Do you think it would be better to write both in same style?

<ol>

 <li>Trace the call musl_memmove(NULL, “cs107”, 0) . Will it result in a segmentation fault from trying to read/write an invalid pointer? Why or why not? What about the call musl_memmove(NULL, “cs107”, -1) ? Verify your understanding by running the code</li>

</ol>

program.

<ol>

 <li>Use make clean and make to see the compiler warnings from the code.c program. The complaints you get are warnings targeted at certain invalid calls to the standard</li>

</ol>

memmove / memcpy functions. The raw memory functions are necessarily de ned with an

extremely permissive interface (any two pointers and size will do) which makes them ripe for misuse. These warnings catch only a few easily-identi ed errors, there are many more errors that won’t be reported, but it’s interesting to note that gcc thought it important enough to make a special case out of them.

The implementation of memmove should remind you of the strncpy function you studied back in lab2 (/class/cs107/lab2). The memxxx functions have much in common with their strxxx equivalents, just without the special case to stop at a null byte. In fact, the memxxx functions are declared as part of the &lt;string.h&gt; module and quite possibly written by the same author.

<h3>Comparison functions</h3>

The sort/search functions of the C standard library are written as generics (e.g. using void* ) and require the client to supply a callback comparison function to compare elements. A comparison function often only needs simple logic to do its task, but managing the syntax and applying the correct level of indirection is where the trickiness comes in.

The client’s comparison function must t the standard comparison function

(https://www.gnu.org/software/libc/manual/html_node/Comparison-

Functions.html#Comparison-Functions) as described in the GNU libc manual. I lightly paraphrased its example comparison function below:

One essential feature to note is that the arguments to a comparison function are two <strong>pointers to the elements</strong> to be compared, <strong>not the values</strong> of those elements. As a rule, the rst thing a callback function should do is to cast the incoming void* arguments and copy into the variables of the proper type. The rest of the function can then operate on those properly-typed variables and receive the bene t of the compiler’s help in respecting type safety, which you do not get if directly accessing the void* .

A more minor detail is that it is only the sign of the comparison function result that matters. For example, -1 or any negative result from the comparison function is taken as an indication that the rst element is less than the second.

Look back at the compare_doubles function above. Work out these questions with your partner:

<ol>

 <li>What is the result of evaluating any inequality such as first &gt; second ? (Try it in gdb to con rm!) Interesting, the relational/equality operators are de ned to produce a value of either 0 or 1 and “clever” programmers are wont to take advantage of this.</li>

 <li>How does the subtraction of the two inequalities compute a result of the correct sign?</li>

 <li>The more obvious approach would be a chained if/else of three separate cases for</li>

</ol>

less/equal/greater. That version would certainly be more readable and easier to verify

correct. What do you think might have motivated the author instead to write it as shown above?

<h3>Comparing ints</h3>

Your colleague checks in a comparison function for integer elements into your team’s repo. The implementation subtracts the two values and uses their di erence as the comparison result.

During code review, you point out to your colleague that this function will not work correctly in case of over ow. For example, if the rst element is a large enough positive value and second a large enough negative value such that their di erence exceeds INT_MAX , the function result will be incorrect, causing those two elements to be incorrectly ordered.

<ol>

 <li>Use the ints program to observe the consequence of the over ow bug. The test program puts a few extreme values in the array (to deliberately trigger over ow) and randomizes the remaining array elements. It qsorts the array using the above comparison function. Run the program several times. You should observe that the extreme values end up in wacky places, while the other elements are mostly sorted correctly.</li>

</ol>

Your colleague concurs with your bug report when you demonstrate this reproducible failure. To x, he proposes to avoid the over ow by promoting the values to the larger bitwidth long type before the subtraction:

Discuss with your partner:

<ol>

 <li>Change the ints program to use this version of the comparison function and try running it several times. This proposed x does absolutely nothing in terms of correcting the error. Why not?</li>

</ol>

<h3>Comparing structs</h3>

Let’s do a quick recap of structs before our next example: C struct declarations are almost, but not exactly, the same as C++. In C, the following declares a new struct type:

The name of the type is struct coord and you cannot drop the struct keyword; declaring a variable of type coord will not compile.

The . (dot) operator is used to access the elds within a struct. If you have a pointer to a struct, your rst attempt to access the elds is likely to run afoul of the fact that . has higher precedence than * . You can add parentheses to force the desired precedence, or better, use the -&gt; operator which combines . and * for this common need.

struct coord origin;                            // struct on stack struct coord *p = malloc(sizeof(struct coord)); // struct in heap                    // (note: sizeof works correctly for structs)




origin.x = 0;  // access field from struct variable




// these next 3 lines access field via struct pointer

*p.x = 0;      // WRONG! precedence applies . first then * (*p).x = 0;    // OK: parens used to override precedence p-&gt;x = 0;      // BEST: preferred way to access

The searchsort.c le in the starter code contains the example sort-search program

(https://www.gnu.org/software/libc/manual/html_node/Search_002fSort-

Example.html#Search_002fSort-Example) from the GNU libc manual. The example demonstrates using the qsort and bsearch library functions on an array consisting of elements of struct type.

<ol>

 <li>Review the program to see a struct de nition and its use. Note how the generic functions can support this new type by virtue of the client-supplied comparison function. The comparison function orders the struct critters by comparing the name eld. Two critters of the same name would be considered equal by the function.</li>

 <li>As an exercise in working with callback functions, edit the comparison function to break</li>

</ol>

ties between critters of the same name by comparing their species.

<h2>2) gdb protip: printing arrays</h2>

One handy gdb feature we want to get into your repertoire is how to print arrays. If you print a stack array from within the function it is declared, gdb will show the array and its contents. In that context, gdb has access to both the element type and the count of elements, and uses it to print a

nice representation of the entire array. However it cannot automatically do the same in other contexts, such as for a heap array or for an array/pointer passed into a function. It is possible to print the entire array in those contexts, but you have to provide more information to gdb. Let’s see how!

<ol>

 <li>Run gdb on the program ints . Set a breakpoint on main and step into the function past the variable declaration/initializations.</li>

 <li>Try p nums . Here in the context of its declaration, gdb knows that it is a stack array of a certain size and can show the entire stack array. Great!</li>

 <li>Now try p argv . All gdb knows about argv is that it is a pointer. Bummer.</li>

 <li>Try p argv[0]@argc and gdb will now print the entire contents of the argv array. Hooray!</li>

 <li>The syntax to learn is p <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="8dc8c1c8c0cdcec2d8c3d9">[email protected]</a> where ELEM is the 0th element and COUNT is the count of elements to print. ELEM and COUNT are C expressions and can refer to any variables in current scope. Try p nums[1]@2 to show a 2-element portion in the middle of the array.</li>

 <li>You can also add in a typecast if needed. For example, given a parameter ptr of type void* that you know is the base address of an array of nelems elements of type char* ,</li>

</ol>

you could print the entire array as p *(char **)<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="c8b8bcba88a6ada4ada5bb">[email protected]</a> .

<h2>3) Write, test, debug, repeat</h2>

Now it’s your turn! The dups.c program is intended to process an array of numbers given as command-line arguments and report whether the array contains any duplicate values and then remove those duplicates.

<ol>

 <li>Read over the given code to see what you start with. The has_duplicates function is correctly implemented and models the standard technique to iterate over a generic array and call the client’s callback function. Be sure you understand this code before going further!</li>

 <li>The program is missing the implementation of the integer comparison function. The program intends to use a cmp_magnitude function that compares elements by their absolute value, i.e. -5 is &gt; 0, -7 == 7, and 1 &lt; -2. Rather than start with the buggy int comparison looked at earlier in this lab, write your own simple version from scratch that returns a correctly signed result based on comparing the absolute value of the two elements.</li>

 <li>The program is also missing the implementation of the remove_duplicates Use the approach shown in has_duplicates to nd duplicate elements within the generic array and upon nding one, remove it from the array by moving the neighboring elements down by one position and decrementing the array’s element count. The entire block of elements should be moved in one call; this is much more e cient than a loop that copies the elements one by one. You must use memmove , not memcpy , in this situation — why?</li>

 <li>Using sanitycheck to verify the correctness of your work.</li>

</ol>

<h1>Check o with TA</h1>

Before you leave, complete your checko form and ask your lab TA to approve it so you are properly credited. If you don’t complete all the exercises during the lab period, we encourage you to followup and nish the remainder on your own. Try our self-check

(/class/cs107/selfcheck.html#lab4) to re ect on what you’ve done and how it’s going.


