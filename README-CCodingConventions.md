0.	The overarching rule: follow the conventions in the file to which you are making modifications.

1.	Code indented with tabs, not spaces. If you use `vim`, set your `.vimrc` as follows so you can see where you have stray spaces:
	```
	set list
	set listchars=tab:>-
	```
	For visual alignment, assume the code will be viewed in an editor where tabs are rendered to be as wide as eight spaces.

2.	Variable names in `camelCase`. In a time when displays were 80 characters wide by 25 characters high, it made sense to keep variable names short. Not any longer: avoid abbreviating variable names. The abbreviations that seem obvious to you could be baffling (or offensive) to others.

3.	Function return types on a separate line preceding the function head:
	```c
	int
	main(void)
	{
		return 0;
	}
	```

4.	Variable declations with type separated from variable names by a tab and float all variable definitions to the top of the enclosing scope, separated by no empty lines unless necessary to denote groups of different variable classes:
	```c
	int	myVariable;
	void *	myPointer;

	int
	myFunction
	{
		double	myRealNumber;
		char *	myString;
		...
	```

5.	For multi-word type specifiers (e.g., `char *`), leave more space between the type specifier and the identifier, than between the type specifier's components. Recommend using two (or more) spaces in function signatures and a tab in variable declations as above. So, e.g., `char *   argv[]` and not `char * argv[]`

6.	Type names begin with a capital. Type names specific to a project named Xyz begin with `Xyz`, e.g., `XyzPhenomenon`.

7.	Similarly, constant names and enum entries begin with `kXyz`, e.g., `kWarpPhenomenonTemperatureCelcius`. 

8.	C-style comments, in the form:
	```c
		/*
		 *	Comment (offset with a single tab) :-)
		 */

		/*
		 * This comment is offset with a space, not with a tab :-(
		 */
	```

9.	Please avoid C++-style `//` comments unless they are for temporary debugging comments (e.g., temporarily commenting out a line of code)

10.	Comments are not just "notes to self". They should provide useful explanatory information.

11.	No `#include` within header .h files if possible.

12.	No function definitions in header .h files.

13.	For files on project Xyz, file names `xyz-camelCasedName.yyy`. See the [README-FileNamingConventions.md](https://github.com/physical-computation/Conventions/blob/master/README-FileNamingConventions.md) for more on file naming conventions. 

14.	Constants in `enum`s, not in `#define`s where possible. `#define` constants are invisible to the debugger since they are textually substituted by the preprocessor. `enum`s on the other hand are visible to the compiler and go into the debug symbols. Whenever possible, `typedef` the enum to introduce a typename that is initial capital. The members of an enum are constants and should begin with `k` followed by the typename as prefix. An example from the Warp firmware:
	```c
	typedef enum
	{
		kWarpPowerModeWAIT,
		kWarpPowerModeSTOP,
		kWarpPowerModeVLPR,
		kWarpPowerModeVLPW,
		kWarpPowerModeVLPS,
		kWarpPowerModeVLLS0,
		kWarpPowerModeVLLS1,
		kWarpPowerModeVLLS3,
		kWarpPowerModeRUN,
	} WarpPowerMode;
	```
	In the example above, the typename for the enum is initial capitalized (`WarpPowerMode`) and the constants begin with the typename as a prefix (`kWarpPowerModeVLLS3`). Don't confuse constants (defined in enums) with variables: do not name a variable something like `kMyVarName`.

15.	Avoid `#define` if possible (see above). Avoid conditional compilation via `#ifdef` unless there really is a need for code to be compiled out rather than having conditional execution via an `if` statement (e.g., in the Warp firmware to save space).

16.	All `if` statement followed by curly braces, even if body is a single statement.

17.	Curly brace on line following `if`, `for`, function bodies, etc:
	```c
	int
	main(int argc, char *  argv[])
	{
		if (argc == 0)
		{
		}
	}
	```

18.	The patterns ` \n` (space followed by newline) and `\t\n` (tab followed by newline) should never occur in a source file.

19.	For projects that use the `libflex` library (https://github.com/phillipstanleymarbell/libflex), except for temporary debugging statements, all console output statements should use `flexprint`. This allows us to buffer print statements and makes the web interface/demos and other deployments possible. Errors go into the buffer `Fperr` and informational output (almost everything that is not an error) goes into `Fpinfo`. We sometimes have additional dedicated buffers to isolate certain outputs.

20.	Avoid _magic numbers_ (unnamed constants inline in code). Any integer constants used in code should be in an enum (see `enum` entry naming guidelines above).

21.	Here is a long example from the Noisy compiler:
	```c
	/*
	 *	kNoisyIrNodeType_PmoduleDecl
	 *
	 *	Grammar production:
	 *		moduleDecl	::=	identifier ":" "module" "(" typeParameterList ")" "{" moduleDeclBody "}" .
	 *
	 *	Generated AST subtree:
	 *
	 *		node.left	= kNoisyIrNodeType_Tidentifier
	 *		node.right	= Xseq of kNoisyIrNodeType_PtypeParameterList and kNoisyIrNodeType_PmoduleDeclBody
	 */
	IrNode *
	noisyParseModuleDecl(State *  N, Scope *  scope)
	{
		TimeStampTraceMacro(kNoisyTimeStampKeyParseModuleDecl);

		IrNode *	n = genIrNode(N,	kNoisyIrNodeType_PmoduleDecl,
							NULL /* left child */,
							NULL /* right child */,
							lexPeek(N, 1)->sourceInfo /* source info */);
		IrNode *	identifier = noisyParseIdentifierDefinitionTerminal(N, scope);

		addLeaf(N, n, identifier);
		noisyParseTerminal(N, kNoisyIrNodeType_Tcolon);
		noisyParseTerminal(N, kNoisyIrNodeType_Tmodule);
		noisyParseTerminal(N, kNoisyIrNodeType_TleftParens);
		addLeafWithChainingSeq(N, n, noisyParseTypeParameterList(N, scope));
		noisyParseTerminal(N, kNoisyIrNodeType_TrightParens);
	...
		if (!inFollow(N, kNoisyIrNodeType_PmoduleDecl, gNoisyFollows, kNoisyIrNodeTypeMax))
		{
			noisyParserSyntaxError(N, kNoisyIrNodeType_PmoduleDecl, kNoisyIrNodeTypeMax, gNoisyFollows);
			noisyParserErrorRecovery(N, kNoisyIrNodeType_PmoduleDecl);
		}

		return n;
	}
	```

22. We usually include an authorship notice and disclaimer of the following form
	```c
	/*
		Authored 2020, Jane Doe.

		All rights reserved.

		Redistribution and use in source and binary forms, with or without
		modification, are permitted provided that the following conditions
		are met:

		*	Redistributions of source code must retain the above
			copyright notice, this list of conditions and the following
			disclaimer.

		*	Redistributions in binary form must reproduce the above
			copyright notice, this list of conditions and the following
			disclaimer in the documentation and/or other materials
			provided with the distribution.

		*	Neither the name of the author nor the names of its
			contributors may be used to endorse or promote products
			derived from this software without specific prior written
			permission.

		THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
		"AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
		LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS
		FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE
		COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT,
		INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING,
		BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
		LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
		CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT
		LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN
		ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
		POSSIBILITY OF SUCH DAMAGE.
	*/

	```
