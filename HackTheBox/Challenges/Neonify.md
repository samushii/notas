
the main page has an input form that transforms the input into a neon font

looking through the source files, in app/controllers/neon.rb there is a regex that only matches alphanumeric characters

in app/views/index.erb there is a template being used (*@neon*) which is exactly the text passing through the regex aforementioned

ruby has the default behaviour of only matching until a new line if ^ and $ (\\A and \\Z match the whole string and should be preferred, so it is possible to craft a string to read the contenxt of file.txt

	a%0a<%= File.read("flag.txt") %>
[%0a is the url encoded LF ASCII character]