the challenge description tells of a diagnostic.doc on the server, downloading and unziping it will reveal some folders

analyzing the archives using *oletools* [a python library to analyze microsoft ole2 and ms office files], more specifically using *olevba* to check for VBA macros will reveal a suspicuiys url in the file **_rels/document.xml.rels** 

accessing it and analyzing the source code will reveal an obfuscated payload, converting the base64 will reveal a powershell command that uses -f strings to assemble valid expressions

the flag is present in the first -f statement
