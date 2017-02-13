# HashDiff [![Build Status](https://secure.travis-ci.org/liufengyun/hashdiff.png)](http://travis-ci.org/liufengyun/hashdiff) [![Gem Version](https://badge.fury.io/rb/hashdiff.png)](http://badge.fury.io/rb/hashdiff)

HashDiff is a ruby library to compute the smallest difference between two hashes.

**Docs**: [Documentation](http://rubydoc.info/gems/hashdiff)

## Usage

To use the gem, add the following to your Gemfile:

```ruby
gem 'hashdiff'
```

## Quick Start

### Diff

Two simple hashes:

```ruby
a = {a:3, b:2}
b = {}

diff = HashDiff.diff(a, b)
diff.should == [
 ['-', 'a', 3], 
 ['-', 'b', 2]
]

```

Hashes from real life of dev:

```ruby
a = {
 :id=>"16b8d68f-b173-42f8-b6c9-4b0b2893dc0d",
 :code=>"WZX",
 :contract_id=>"542354543",
 :account=>"654364536543",
 :name_id=>"7c32404c-0588-417e-9eac-416dce6a095f",
 :supplier_id=>"31e63dce-0ec7-45d8-9da9-1fd952ca3aa6",
 :season_id=>"f52235c1-3fdb-46a4-9038-31117db0e56f",
 :name=>"Name",
 :supplier=>"NEW",
 :profile=>nil,
 :services=>[],
 :geolocations=>[]
}

b = {
 :id=>"16b8d68f-b173-42f8-b6c9-4b0b2893dc0d",
 :code=>"WZX",
 :contract_id=>"43243432",
 :account=>"54325432",
 :name_id=>"7c32404c-0588-417e-9eac-416dce6a095f",
 :supplier_id=>"31e63dce-0ec7-45d8-9da9-1fd952ca3aa6",
 :season_id=>"f52235c1-3fdb-46a4-9038-31117db0e56f",
 :name=>"Name",
 :supplier=>"NEW",
 :profile=>nil,
 :services=>["Weight", "Weight"],
 :geolocations=>["Germany", "Germany"]
}

diff = HashDiff.diff(a, b)
diff.should == [
 ["~", "account", "654364536543", "54325432"],
 ["~", "contract_id", "542354543", "43243432"],
 ["+", "geolocations[0]", "Germany"],
 ["+", "geolocations[1]", "Germany"],
 ["+", "services[0]", "Weight"],
 ["+", "services[1]", "Weight"]
]
```


### Patch

patch example:

```ruby
a = {a: 3}
b = {a: {a1: 1, a2: 2}}

diff = HashDiff.diff(a, b)
HashDiff.patch!(a, diff).should == b
```

unpatch example:

```ruby
a = [{a: 1, b: 2, c: 3, d: 4, e: 5}, {x: 5, y: 6, z: 3}, 1]
b = [1, {a: 1, b: 2, c: 3, e: 5}]

diff = HashDiff.diff(a, b) # diff two array is OK
HashDiff.unpatch!(b, diff).should == a
```

### Options

There are six options available: `:delimiter`, `:similarity`, `:strict`, `:numeric_tolerance`, `:strip` and `:case_insensitive`.

#### `:delimiter`

You can specify `:delimiter` to be something other than the default dot. For example:

```ruby
a = {a:{x:2, y:3, z:4}, b:{x:3, z:45}}
b = {a:{y:3}, b:{y:3, z:30}}

diff = HashDiff.diff(a, b, :delimiter => '\t')
diff.should == [['-', 'a\tx', 2], ['-', 'a\tz', 4], ['-', 'b\tx', 3], ['~', 'b\tz', 45, 30], ['+', 'b\ty', 3]]
```

#### `:similarity`

In cases where you have similar hash objects in arrays, you can pass a custom value for `:similarity` instead of the default `0.8`.  This is interpreted as a ratio of similarity (default is 80% similar, whereas `:similarity => 0.5` would look for at least a 50% similarity).

#### `:strict`

The `:strict` option, which defaults to `true`, specifies whether numeric types are compared on type as well as value.  By default, a Fixnum will never be equal to a Float (e.g. 4 != 4.0).  Setting `:strict` to false makes the comparison looser (e.g. 4 == 4.0).

#### `:numeric_tolerance`

The :numeric_tolerance option allows for a small numeric tolerance.

```ruby
a = {x:5, y:3.75, z:7}
b = {x:6, y:3.76, z:7}

diff = HashDiff.diff(a, b, :numeric_tolerance => 0.1)
diff.should == [["~", "x", 5, 6]]
```

#### `:strip`

The :strip option strips all strings before comparing.

```ruby
a = {x:5, s:'foo '}
b = {x:6, s:'foo'}

diff = HashDiff.diff(a, b, :comparison => { :numeric_tolerance => 0.1, :strip => true })
diff.should == [["~", "x", 5, 6]]
```

#### `:case_insensitive`

The :case_insensitive option makes string comparisions ignore case.

```ruby
a = {x:5, s:'FooBar'}
b = {x:6, s:'foobar'}

diff = HashDiff.diff(a, b, :comparison => { :numeric_tolerance => 0.1, :case_insensitive => true })
diff.should == [["~", "x", 5, 6]]
```

#### Specifying a custom comparison method

It's possible to specify how the values of a key should be compared.

```ruby
a = {a:'car', b:'boat', c:'plane'}
b = {a:'bus', b:'truck', c:' plan'}

diff = HashDiff.diff(a, b) do |path, obj1, obj2|
  case path
  when  /a|b|c/
    obj1.length == obj2.length
  end
end

diff.should == [['~', 'b', 'boat', 'truck']]
```

The yielded params of the comparison block is `|path, obj1, obj2|`, in which path is the key (or delimited compound key) to the value being compared. When comparing elements in array, the path is with the format `array[*]`. For example:

```ruby
a = {a:'car', b:['boat', 'plane'] }
b = {a:'bus', b:['truck', ' plan'] }

diff = HashDiff.diff(a, b) do |path, obj1, obj2|
  case path
  when 'b[*]'
    obj1.length == obj2.length
  end
end

diff.should == [["~", "a", "car", "bus"], ["~", "b[1]", "plane", " plan"], ["-", "b[0]", "boat"], ["+", "b[0]", "truck"]]
```

When a comparison block is given, it'll be given priority over other specified options. If the block returns value other than `true` or `false`, then the two values will be compared with other specified options.

#### Sorting arrays before comparison

An order difference alone between two arrays can create too many diffs to be useful. Consider sorting them prior to diffing.

```ruby
a = {a:'car', b:['boat', 'plane'] }
b = {a:'car', b:['plane', 'boat'] }

HashDiff.diff(a, b) => [["+", "b[0]", "plane"], ["-", "b[2]", "plane"]]

b[:b].sort!

HashDiff.diff(a, b) => []
```

## License

HashDiff is distributed under the MIT-LICENSE.

