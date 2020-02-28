1.	Code indented with tabs, not spaces. If you use `vim`, set your `.vimrc` as follows so you can see where you have stray spaces:
```
set list
set listchars=tab:>-
```
For visual alignment, assume the code will be viewed in an editor where tabs are rendered to be as wide as eight spaces.

2.	Variable names in `camelCase`.

3.	Function return types on a separate line preceding the function head:
```c
int
main(void)
{
	return 0;
}
```

4.	Variable declations with type separated from variable names by a tab:
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

5.	For multi-word type specifiers (e.g., `char *`), leave more space between the type specifier and the identifier, than between the type specifier's components. Recommend using two spaces in function signatures and a tab in variable declations as above. So, e.g., `char *  argv[]` and not ``char * argv[]`

6.	Type names begin with a capital. Type names specific to a project named Xyz begin with `Xyz`, e.g., `XyzPhenomenon`.

7.	Similarly, constant names and enum entries begin with `kXyz`, e.g., `kWarpPhenomenonTemperatureCelcius`. 

8.	C-style comments, in the form:
```c
	/*
	 *	Comment (offset with a single tab)
	 */
```

	Comment content offset with a tab, so _not_

```c
	/*
	 * This comment is offset with a space, not with a tab.
	 */
```

	Please avoid C++-style `//` comments unless they are for temporary debugging comments (e.g., temporarily commenting out a line of code)

9.	Comments are not just "notes to self". They should provide useful explanatory information.

10.	No `#include` within header .h files if possible.

11.	No function definitions in header .h files.

12.	For files on project Xyz, file names `xyz-camelCasedName.yyy`. See the [README-FileNamingConventions.md](https://github.com/physical-computation/Conventions/blob/master/README-FileNamingConventions.md) for more on file naming conventions. 

13.	Constants in `enum`s, not in `#define`s where possible.

14.	Avoid `#define` if possible.

15.	All `if` statement followed by curly braces, even if body is a single statement.

16.	Curly brace on line following `if`, `for`, function bodies, etc:
```C
int
main(int argc, char *  argv[])
{
	if (argc == 0)
	{
	}
}
```

17.	The patterns ` \n` (space followed by newline) and `\t\n` (tab followed by newline) should never occur in a source file.

18.	Except for temporary debugging statements, all print statements should use `flexprint` from the `libflex` library (https://github.com/phillipstanleymarbell/libflex). This allows us to buffer print statements and makes the web interface/demos and other deployments possible. Errors go into the buffer `Fperr` and informational output (almost everything that is not an error) goes into `Fpinfo`. We sometimes have additional dedicated buffers to isolate certain outputs.

19. Here is a long example from the Noisy compiler:
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

	/*
	 *	We keep a global handle on the module scope
	 */
	IrNode *	scopeBegin	= noisyParseTerminal(N, kNoisyIrNodeType_TleftBrace);
	Scope *		moduleScope	= commonSymbolTableOpenScope(N, scope, scopeBegin);
	IrNode *	typeTree	= noisyParseModuleDeclBody(N, moduleScope);

	addLeaf(N, n, typeTree);

	IrNode *	scopeEnd	= noisyParseTerminal(N, kNoisyIrNodeType_TrightBrace);

	commonSymbolTableCloseScope(N, moduleScope, scopeEnd);
	identifier->symbol->typeTree = typeTree;

	addToModuleScopes(N, identifier->symbol->identifier, moduleScope);

	if (!inFollow(N, kNoisyIrNodeType_PmoduleDecl, gNoisyFollows, kNoisyIrNodeTypeMax))
	{
		noisyParserSyntaxError(N, kNoisyIrNodeType_PmoduleDecl, kNoisyIrNodeTypeMax, gNoisyFollows);
		noisyParserErrorRecovery(N, kNoisyIrNodeType_PmoduleDecl);
	}

	return n;
}
```
