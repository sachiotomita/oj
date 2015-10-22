# Oj gem
A fast JSON parser and Object marshaller as a Ruby gem.

## Using

To use oj as a standalone piece you'll probably just use these functions:

``` ruby
native = {
  "person" => {
    "name" => "President Commancho",
    "age" => 23
    "likes" => [
      "guns",
      "america",
      "motorcycles"
      "not sure"
    ]
  }
}
json = Oj.dump(native)
# => (json dump)

native == Oj.load(json)
```

There are some extra options you can use:

``` ruby
Oj.default_options.merge({
  indent: Fixnum,
  # number of spaces to indent each element in an JSON document, zero is no newline between JSON elements, negative indicates no newline between top level JSON elements in a stream
  
  circular: Boolean || nil,
  # support circular references while dumping
  
  auto_define: Boolean || nil,
  # automatically define classes if they do not exist
  
  symbol_keys: Boolean || nil,
  # use symbols instead of strings for hash keys
  
  escape_mode: :newline || :json || :xss_safe || :ascii || nil,
  # determines the characters to escape
  
  class_cache: Boolean || nil,
  # cache classes for faster parsing (if dynamically modifying classes or reloading classes then don't use this)
  
  mode: :object || :strict || :compat || :null,
  # load and dump modes to use for JSON
  
  time_format: :unix || :unix_zone || :xmlschema || :ruby,
  # time format when dumping in :compat and :object mode
  
  bigdecimal_as_decimal: Boolean || nil,
  # dump BigDecimal as a decimal number or as a String
  
  bigdecimal_load: :bigdecimal || :float || :auto,
  # load decimals as BigDecimal instead of as a Float. :auto pick the most precise for the number of digits.
  
  create_id: String || nil,
  # create id for json compatible object encoding, default is 'json_create'
  
  second_precision: Fixnum || nil,
  # number of digits after the decimal when dumping the seconds portion of time
  
  float_precision: Fixnum || nil,
  # number of digits of precision when dumping floats, 0 indicates use Ruby
  
  use_to_json: Boolean || nil,
  # call to_json() methods on dump, default is false
  
  nilnil: Boolean || nil,
  # if true a nil input to load will return nil and not raise an Exception
  
  allow_gc: Boolean || nil,
  # allow or prohibit GC during parsing, default is true (allow)
  
  quirks_mode: Boolean || nil
  # Allow single JSON values instead of documents, default is true (allow)
})
```


## Installation
```
gem install oj
```
or in Bundler:
```
gem 'oj'
```

## Documentation

*Documentation*: http://www.ohler.com/oj, http://rubydoc.info/gems/oj

## Source

*GitHub* *repo*: https://github.com/ohler55/oj

*RubyGems* *repo*: https://rubygems.org/gems/oj

Follow [@peterohler on Twitter](http://twitter.com/#!/peterohler) for announcements and news about the Oj gem.

## Build Status

[![Build Status](https://secure.travis-ci.org/ohler55/oj.png?branch=master)](http://travis-ci.org/ohler55/oj)

## Release 2.13.0

 - Oj no longer raises an exception if the to_hash method of an object does not
   return a Hash. ActiveRecord has decided that to_hash should return an Array
   instead so Oj now encodes what ever is returned.

 - Added a register_odd_raw function that allows odd marshal functions to return
   raw JSON as a string to be included in the dumped JSON.

 - The register_odd function now supports modules in additions to classes.

[Older release notes](http://www.ohler.com/dev/oj_misc/release_notes.html).

## Description

Optimized JSON (Oj), as the name implies, was written to provide speed optimized
JSON handling. It was designed as a faster alternative to Yajl and other
common Ruby JSON parsers. So far it has achieved that, and is about 2 times faster
than any other Ruby JSON parser, and 3 or more times faster at serializing JSON.

Oj has several `dump` or serialization modes which control how Ruby `Object`s are
converted to JSON. These modes are set with the `:mode` option in either the
default options or as one of the options to the `dump` method. The default mode
is the `:object` mode.

- `:strict` mode will only allow the 7 basic JSON types to be serialized. Any
  other `Object` will raise an `Exception`.

- `:null` mode replaces any `Object` that is not one of the JSON types with a JSON `null`.

- `:object` mode will dump any `Object` as a JSON `Object` with keys that match the
  Ruby `Object`'s variable names without the '@' prefix character. This is the highest
  performance mode.

- `:compat` mode attempts to be compatible with other systems. It will serialize any
  `Object`, but will check to see if the `Object` implements an `to_hash` or `to_json`
  method. If either exists, that method is used for serializing the `Object`.
  Since `as_json` is more flexible and produces more consistent output, it is
  preferred over the `to_json` method. If neither the `to_json` or `to_hash`
  methods exists, then the Oj internal `Object` variable encoding is used.

To change default serialization mode:
```ruby
Oj.default_options = {:mode => :compat }
```

## Compatibility

### Ruby
Oj is compatible with Ruby 1.8.7, 1.9.2, 1.9.3, 2.0.0, 2.1, 2.2 and RBX.
Support for JRuby has been removed as JRuby no longer supports C extensions and
there are bugs in the older versions that are not being fixed.

### Rails
Although up until 4.1 Rails uses [multi_json](https://github.com/intridea/multi_json), an [issue in Rails](https://github.com/rails/rails/issues/9212) causes ActiveSupport to fail to make use Oj for JSON handling.
There is a
[gem to patch this](https://github.com/GoodLife/rails-patch-json-encode) for
Rails 3.2 and 4.0. As of the Oj 2.6.0 release the default behavior is to not use
the `to_json()` method unless the `:use_to_json` option is set. This provides
another work around to the rails older and newer behavior.

The latest ActiveRecord is able to work with Oj by simply using the line:
```
serialize :my_attr, Oj
```

In version Rails 4.1, multi_json has been removed, and this patch is unnecessary and will no longer work.
Instead, use the `oj_mimic_json` [gem](https://github.com/ohler55/oj_mimic_json) along with `oj` in your `Gemfile` to have Oj mimic the JSON gem and be used in its place by `ActiveSupport` JSON handling:
```
gem 'oj'
gem 'oj_mimic_json'
```

## Proper Use

Two settings in Oj are useful for parsing but do expose a vulnerability if used from an untrusted source. Symbolized
keys can cause memory to be filled with previous versions of ruby. Ruby 2.1 and below does not garbage collect Symbols. The same is true for auto
defining classes in all versions of ruby; memory will also be exhausted if too many classes are automatically defined. Auto defining is a useful
feature during development and from trusted sources but it allows too many classes to be created in the object load
mode and auto defined is used with an untrusted source. The `Oj.strict_load()` method sets and uses the most strict and safest options. It should be used by developers who find it difficult to understand the options available in Oj.

The options in Oj are designed to provide flexibility to the developer. This flexibility allows Objects to be serialized
and deserialized. No methods are ever called on these created Objects but that does not stop the developer from calling
methods on them. As in any system, check your inputs before working with them. Taking an arbitrary `String`
from a user and evaluating it is never a good idea from an unsecure source. The same is true for `Object` attributes as
they are not more than `String`s. Always check inputs from untrusted sources.


## Simple JSON Writing and Parsing Example

```ruby
require 'oj'

h = { 'one' => 1, 'array' => [ true, false ] }
json = Oj.dump(h)

# json =
# {
#   "one":1,
#   "array":[
#     true,
#     false
#   ]
# }

h2 = Oj.load(json)
puts "Same? #{h == h2}"
# true
```

## Alternative JSON Processing APIs

Oj offers a few alternative APIs for processing JSON. The fastest one is the `Oj::Doc` API. The `Oj::Doc` API takes a
completely different approach by opening a JSON document and providing calls to navigate around the JSON while it is
open. With this approach, JSON access can be well over 20 times faster than conventional JSON parsing.

The `Oj::Saj` and `Oj::ScHandler` APIs are callback parsers that
walk the JSON document depth first and makes callbacks for each element.
Both callback parser are useful when only portions of the JSON are of
interest. Performance up to 20 times faster than conventional JSON is
possible. The API is simple to use but does require a different approach than
the conventional parse followed by access approach used by conventional JSON
parsing.


# Links

## Performance Comparisons

[Oj Strict Mode Performance](http://www.ohler.com/dev/oj_misc/performance_strict.html) compares Oj strict mode parser performance to other JSON parsers.

[Oj Compat Mode Performance](http://www.ohler.com/dev/oj_misc/performance_compat.html) compares Oj compat mode parser performance to other JSON parsers.

[Oj Object Mode Performance](http://www.ohler.com/dev/oj_misc/performance_object.html) compares Oj object mode parser performance to other marshallers.

[Oj Callback Performance](http://www.ohler.com/dev/oj_misc/performance_callback.html) compares Oj callback parser performance to other JSON parsers.

## Links of Interest

*Fast XML parser and marshaller on RubyGems*: https://rubygems.org/gems/ox

*Fast XML parser and marshaller on GitHub*: https://github.com/ohler55/ox

[Oj Object Encoding Format](http://www.ohler.com/dev/oj_misc/encoding_format.html) describes the OJ Object JSON encoding format.

[Need for Speed](http://www.ohler.com/dev/need_for_speed/need_for_speed.html) for an overview of how Oj::Doc was designed.

*OjC, a C JSON parser*: https://www.ohler.com/ojc also at https://github.com/ohler55/ojc
