---
layout: post
title: "Rails index-by, index-with"
author: "Sebastián Palma"
---


##### Enumerable#index_by



There are two methods in the Ruby on Rails framework that got my attention when I heard about them for the first time; `Enumerable#index_by` and `Enumerable#index_with`.

While the first one is already more than 14 years old (added by [@seckar](https://github.com/seckar) in June 2006 - see [#a552651](https://github.com/rails/rails/commit/a55265132b37c6fb8ac15a96b44e64a64bcd4c45)), the other one was just introduced a couple of years ago. And they've remained untouched several years, just for a few performance updates, but clearly there's not much to do since the code is pretty simple:



```ruby
def index_by
  if block_given?
    result = {}
    each { |elem| result[yield(elem)] = elem }
    result
  else
    to_enum(:index_by) { size if respond_to?(:size) }
  end
end
```



`index_by` is defined in the `Enumerable` module that Rails provides. It iterates over each element in the receiver assigning the yielded value as the key of a previously instantiated hash named `result` - scoped to the method itself, while the value  is the current element.

Better exemplified (from the docs); you have a `People` class, two simple private attribute readers (`first_name`, `last_name`) and a public method called `login`:



```ruby
class People
  def initialize(first_name, last_name)
    @first_name = first_name
    @last_name = last_name
  end

  def login
    "#{first_name.downcase}-#{last_name.downcase}"
  end

  private

  attr_reader :first_name, :last_name
end

People.new('John', 'Doe').login   # "john-doe"
People.new('Lola', 'Lanos').login # "lola-lanos"
```



Now if you happen to have multiple `people` instances in an `Enumerable` object, and you need to convert their current structure to a Hash, where each key is the current `people` object `login` value, and the value the object itself you can use `index_by` to shorten the amount of characters you'd do in plain Ruby:



```ruby
[People.new('John', 'Doe'), People.new('Lola', 'Lanos')].index_by(&:login)
# {"john-doe"=>#<People:0x00007ff9429640e0 @first_name="John", @last_name="Doe">,
#  "lola-lanos"=>#<People:0x00007ff94381ba48 @first_name="Lola", @last_name="Lanos">}
```



It's a very handy method, as it allows us passing whatever we need through the block and it'll be the value our hash keys take:



```ruby
require 'securerandom'

[People.new('John', 'Doe'), People.new('Lola', 'Lanos')].index_by do |person|
  "#{person.login}-#{SecureRandom.hex(10)}"
end

# {"john-doe-17f0055e009976edb4c5"=>#<People:0x00007f9b50a902e8 @first_name="John", @last_name="Doe">,
#  "lola-lanos-0eee36611ea804227b8e"=>#<People:0x00007f9b50a90270 @first_name="Lola", @last_name="Lanos">}
```



The `securerandom` library is required to generate some random numbers and add an identifier to every person `login`, aiming to make each key unique - as you might know if the same key is set more than once on a `Hash`, the last is the one that preserves. The block code is then yielded inside the `index_by` method, as result every key takes the value of the `login` method invoked on the current object (`self`) interpolated with a random hexadecimal string of 10 digits created using the `SecureRandom` library.

Finally, if the method is invoked without given a block it returns the receiver as an `Enumerator`  which is common when defining custom `Enumerable` methods - from the Ruby docs:



> It is typical to call [#to_enum](https://ruby-doc.org/core-2.7.1/Object.html#method-i-to_enum) when defining methods for a generic [Enumerable](https://ruby-doc.org/core-2.7.1/Enumerable.html), in case no block is passed.



The addition of `index_by` to the core code of Rails aims at making the conversion from one object to another easier to do and to read, but it might not be easy to get the idea without giving it some time to understand it.

There are different ways in which you can do this kind of transformation, like using [`Array#each`](https://ruby-doc.org/core-2.6.5/Array.html#method-i-each) as in the method body of `index_by`, [`Enumerable#map`](https://ruby-doc.org/core-2.6.5/Enumerable.html#method-i-map),  [`Enumerable#each_with_object`](https://ruby-doc.org/core-2.6.5/Enumerable.html#method-i-each_with_object) or even [`Object#tap`](https://ruby-doc.org/core-2.7.1/Object.html#method-i-tap):



```ruby
Hash[people.map { |person| [person.login, person] }]
# {"john-doe"=>#<People:0x00007f8d448a3da0 @first_name="John", @last_name="Doe">, "lola-lanos"=>#<People:0x00007f8d448a3d28 @first_name="Lola", @last_name="Lanos">}
```



This is probably the oldest you can see on internet (besides using `each`). It iterates over each element from the receiver, mapping them as an array where the first element is the result of invoking the `login` method on the current object, and the second and last element is the object itself. After that the result is used to create a new Hash by using the [`Hash::[]`](https://ruby-doc.org/core-2.6.5/Hash.html#method-c-5B-5D) method.



```ruby
people.map { |person| [person.login, person] }.to_h
# {"john-doe"=>#<People:0x00007f8d448a3da0 @first_name="John", @last_name="Doe">, "lola-lanos"=>#<People:0x00007f8d448a3d28 @first_name="Lola", @last_name="Lanos">}
```



You can also see its shorter version using [`Enumerable#to_h`](https://ruby-doc.org/core-2.6.5/Enumerable.html#method-i-to_h) instead of using [`Hash::[]`](https://ruby-doc.org/core-2.6.5/Hash.html#method-c-5B-5D), which makes it look less confusing and probably easier to understand.



```ruby
people.each_with_object({}) { |person, hash| hash[person.login] = person }
# {"john-doe"=>#<People:0x00007f8d448a3da0 @first_name="John", @last_name="Doe">, "lola-lanos"=>#<People:0x00007f8d448a3d28 @first_name="Lola", @last_name="Lanos">}
```



Another common way is to use [`Enumerable#each_with_object`](https://ruby-doc.org/core-2.6.5/Enumerable.html#method-i-each_with_object) and pass an empty Hash as the memo object (`memo_obj`) which is _filled_ by invoking `login` in the current object and assigning that result as one of the `memo_obj` keys, giving the current object as the value for that key. That returns a Hash itself, so it doesn't require any transformation, what makes the approach more clear and maybe the preferred one.



```ruby
{}.tap { |hash| people.each { |person| hash[person.login] = person } }
# {"john-doe"=>#<People:0x00007f8d448a3da0 @first_name="John", @last_name="Doe">, "lola-lanos"=>#<People:0x00007f8d448a3d28 @first_name="Lola", @last_name="Lanos">}
```



As Ruby is flexible enough to leave you explore new things, you can always use [`Object#tap`](https://ruby-doc.org/core-2.7.1/Object.html#method-i-tap) to achieve the same goal; start by _taping_ an empty Hash passing a block where you iterate an object with `each` and do the same procedure as in the previous way but this time modifying the _taped_ object, producing the same expected output.

There's another way I can come up with to get the same result as seen before; [`Enumerable#to_h`](https://ruby-doc.org/core-2.6.5/Enumerable.html#method-i-to_h). `to_h` is since many time the preferred notation to refer to the conversion of one object to a Hash, and since the version 2.6 of Ruby it allows passing a block where you can specify what's the pair of objects in an array that are going to be used during the conversion of the receiver to a Hash:



```ruby
people.to_h { |person| [person.login, person] }
# {"john-doe"=>#<People:0x00007f8d448a3da0 @first_name="John", @last_name="Doe">, "lola-lanos"=>#<People:0x00007f8d448a3d28 @first_name="Lola", @last_name="Lanos">}
```



Four characters less, a single method invoked and a pretty clear intent. A very neat addition for `to_h`.

If you're working with Ruby and your version supports passing a block to `to_h` you can stick with it, otherwise if you're using Rails `index_by` is enough to get it done.

Having so many ways to get the same expected result makes oneself wonder which is the best one. As _best_ is a broad term and depends heavily on the context, the less you can do is to measure them to see which one to choose. For that you can use the [`evanphx/benchmark-ips`](https://github.com/evanphx/benchmark-ips) library - actually Ruby provides `benchmark` which is simpler, if you want you can go with it, the usage is pretty much the same.

If we run a benchmark among all the implementations we've seen we can get something like this:



```ruby
Benchmark.ips do |benchmark|
  benchmark.report('Enumerable#index_by') do
    people.dup.index_by(&:login)
  end

  benchmark.report('Array#each') do
    hash = {}
    people.dup.each { |person| hash[person.login] = person }
    hash
  end

  benchmark.report('Hash::[]') do
    Hash[people.dup.map { |person, hash| [person.login, person] }]
  end

  benchmark.report('Enumerable#map') do
    people.dup.map { |person, hash| [person.login, person] }.to_h
  end

  benchmark.report('Enumerable#each_with_object') do
    people.dup.each_with_object({}) { |person, hash| hash[person.login] = person }
  end

  benchmark.report('Object#tap') do
    {}.tap { |new_hash| people.dup.each { |person, foo| new_hash[person.login] = person } }
  end

  benchmark.report('Enumerable#to_h') do
    people.dup.to_h { |person| [person.login, person] }
  end

  benchmark.compare!
end

...

Comparison:
     Enumerable#to_h:          448486.3 i/s
          Array#each:          418382.0 i/s - same-ish: difference falls within error
 Enumerable#index_by:          386775.4 i/s - same-ish: difference falls within error
          Object#tap:          371543.9 i/s - same-ish: difference falls within error
Enumerable#each_with_object:   357570.7 i/s - same-ish: difference falls within error
      Enumerable#map:          167226.5 i/s - 2.68x  (± 0.00) slower
            Hash::[]:          150846.4 i/s - 2.97x  (± 0.00) slower
```



I'm focusing here just in the comparison. It states that the `to_h` _way_ runs at 448486.3 iterations per second being the fastest one, and in the counterpart it's the `Hash::[]` _way_ running at 150846.4 iterations per second making it almost 3 times slower than `to_h`.

Looking at the result we infer that the difference isn't much, and we could stick our code even to the use of the `each` version, which is by far the less Ruby-_ish_ (for saying something), but we wouldn't be in much troubles because the difference in a scenario like this is almost imperceptible for a human.

`index_by` despite using the same code as in `Array#each` performed less iterations per second, what makes me think it must be something related to have to require the `active_support` library to make it work and all the underlying code regarding to it.

For the ones producing a _"same-ish: difference falls within error"_ output, it means that the difference is of no value. Although different they don't provide a meaningful insight about their performance in comparison to the ones they're paired with.

##### Enumerable#index_with

`index_with` is the younger brother of `index_by` with a similar an interesting functionality. It's meant to be invoked on an `Enumerable` object to return a new hash where each key is the current iterating element in the receiver and the value is the result of yielding the block code with the current object.

Let's exemplify (from the docs as well); a Post class with two public attribute readers `body` and `title`:



```ruby
class Post
  attr_reader :body, :title

  def initialize(body:, title:)
    @body = body
    @title = title
  end
end

post = Post.new(title: "hey there", body: "what's up?")
%i[body title].index_with { |attr_name| post.public_send(attr_name) }
# {:body=>"what's up?", :title=>"hey there"}
```



In the example, the receiver `%i[body title]` is iterated and for each element it assigns to a _temporal_ (kind of `memo_obj`) hash the current element as its key and the result of `post.public_send(attr_name)` as its value, meaning it first uses `body` to assign the hash key:

```ruby
{ :body => post.public_send(attr_name) }
```

And then the result of `post.public_send(attr_name)` is evaluated and set as the hash key/value:

```ruby
{ :body => "what's up?" }
```

Which is the same to do `post.body` if we could, but we can't, because it's meant to be dynamic, and a simple way to do that is by using `public_send`.

The same operation is performed for each element in the receiver. But looking at the code you can notice something interesting:



```ruby
def index_with(default = INDEX_WITH_DEFAULT)
  if block_given?
    result = {}
    each { |elem| result[elem] = yield(elem) }
    result
  elsif default != INDEX_WITH_DEFAULT
    result = {}
    each { |elem| result[elem] = default }
    result
  else
    to_enum(:index_with) { size if respond_to?(:size) }
  end
end
```



If you don't pass a block, it uses the default argument as the value for the current element; that's to say, for all the element in the receiver as long as `default` is different to `Enumerable::INDEX_WITH_DEFAULT`, otherwise it returns the receiver as an `Enumerable` object of `index_with`.

So, the second branch of the condition is pretty useful too. If you want to create a `Hash` from an array with some default values, you can pass it as the first and only one parameter for `index_with`, e.g:



```ruby
%i[body title].index_with(nil)
# {:body=>nil, :title=>nil}

%i[body title].index_with("placeholder")
=> {:body=>"placeholder", :title=>"placeholder"}
```

{% include references.html -%}

##### References

"Ruby on Rails 6.0.3.1 - Enumerable#index_by" - [https://api.rubyonrails.org/classes/Enumerable.html#method-i-index_by](https://api.rubyonrails.org/classes/Enumerable.html#method-i-index_by)
<br /><br />

"Ruby on Rails 6.0.3.1 - Enumerable#index_with" - [https://api.rubyonrails.org/classes/Enumerable.html#method-i-index_with](https://api.rubyonrails.org/classes/Enumerable.html#method-i-index_with)
<br /><br />

"Nicholas Seckar - Add Enumerable#index_by" - [https://github.com/rails/rails/commit/a55265132b37c6fb8ac15a96b44e64a64bcd4c45](https://github.com/rails/rails/commit/a55265132b37c6fb8ac15a96b44e64a64bcd4c45)
<br /><br />

"Kasper Timm Hansen - Add Enumerable#index_with" - [https://github.com/rails/rails/pull/32523](https://github.com/rails/rails/pull/32523)
<br /><br />

"Dmitrii Gautsel - Perfomance fix for Enumerable#index_by" - [https://github.com/rails/rails/commit/6751b1032070a3b26e89a151cbe564a354eb580d](https://github.com/rails/rails/commit/6751b1032070a3b26e89a151cbe564a354eb580d)
<br /><br />

"Ruby Benchmark library" - [https://ruby-doc.org/stdlib-2.5.3/libdoc/benchmark/rdoc/Benchmark.html](https://ruby-doc.org/stdlib-2.5.3/libdoc/benchmark/rdoc/Benchmark.html)
<br /><br />

"BigBinary - Rails 6 adds Enumerable#index_with" - [https://blog.bigbinary.com/2019/06/17/rails-6-adds-enumerable-index_with.html](https://blog.bigbinary.com/2019/06/17/rails-6-adds-enumerable-index_with.html)
<br /><br />

"Saeloun - Rails 6 adds Enumerable#index_with" - [https://blog.saeloun.com/2019/02/24/rails-6-enumerable-index-with.html](https://blog.saeloun.com/2019/02/24/rails-6-enumerable-index-with.html)
<br /><br />

"Georgi Mitrev - Benchmarking Ruby" - [http://mitrev.net/ruby/2015/08/28/benchmarking-ruby/](http://mitrev.net/ruby/2015/08/28/benchmarking-ruby/)
<br /><br />