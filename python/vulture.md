# Vulture and ast

[Vulture](https://bitbucket.org/jendrikseipp/vulture) is a python library that uses the `ast` module to find functions that might never be called in a set of python files. It does this by crawing the code and maintining a set of defined and referenced code objects (considering only their name, not scope). It then reports defined but not used objects. Some interesting things about it:

 * It's a [single file](https://bitbucket.org/jendrikseipp/vulture/src/41e938721fce5ee49d76e3bb2030bafb3acb66a9/vulture.py?fileviewer=file-view-default), fast, and easy to understand.
 * It uses [`ast.NodeVisitor`](https://docs.python.org/2/library/ast.html#ast.NodeVisitor) to walk the `ast`. Dispatch in `NodeVisitor` happens by calling methods named `visit_<NodeType>` with a fallback to `generic_visit`. 
 * It detects references of the form `'%{var}' % locals()`
 