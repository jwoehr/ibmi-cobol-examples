# Jparse COBOL Examples

[jparsez.cblle](jparsez.cblle) is based on a posting on IBM Community
[Using JSON PARSE for json that has sections](https://community.ibm.com/community/user/discussion/using-json-parse-for-json-that-has-sections).

[jparsei.sqlcblle](jparsei.sqlcblle) is a conversion to IBM i COBOL derived from
that original posting.

## Deployment and compilation

Deploy this workspace to your IBM i system to build the examples. Use the
provided [Makefile](Makefile) to compile the programs from a PASE shell (such as
an SSH session).

By default, the `Makefile` builds into the `JWOEHR` object library and expects
the source in `/home/jwoehr/builds/ibmi-cobol-examples/jparse`. You can override
these locations by setting the `OBJLIB` and `SRCDIR` variables when you run
`gmake`.

```bash
gmake OBJLIB=YOURLIB SRCDIR=/path/to/your/deployed/jparse
```
