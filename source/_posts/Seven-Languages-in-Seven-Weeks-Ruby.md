title: Seven Languages in Seven Weeks - Ruby
categories:
  - Study
  - Programming Language
tags:
  - Ruby
  - Seven Languages in Seven Weeks
  - exercise
date: 2015-03-30 22:12:03
---

七周七语言 —— Ruby练习
===

[七周七语言-Io](http://tecton69.com/2015/03/31/Seven-Languages-in-Seven-Weeks-Io/)

我是从网上看到《七周七语言》这本书的，去图书馆借来看了以后发现确实不错。虽然有些人可能觉得这样的介绍书都很简略，但我个人挺喜欢这样的书的，希望它能够帮助我认识之前从来只是听说过却没有了解过的语言——比如Haskel， Erlang， Scala等等。

<!-- more -->

虽然说是七周七语言（ps:网上有人误写作七天七语言（pss:我觉得也是可以的……）），但是其实看看目录就知道每个语言都介绍了3天，最后一个是趁热打铁，即对于这个语言优劣的总结。

做做练习！

###第一天
做

```
1. puts 'Hello, world.'
2. 'Hello, Ruby.'.index('Ruby.')
3. 10.times {puts 'tecton'}
4. Array(1..10).each {|x| puts "This is sentence number #{x}."}
5. ruby xxx.rb
6.  n = rand(10)
    while true
      puts 'Input a number between 1 ~ 9.(inclusive)'
      guess = gets.to_i
      if guess > n
        puts 'Too big!'
      elsif (guess < n)
        puts 'Too small!'
      else
        puts 'Bingo!!!'
        break
      end
    end
```

###第二天
找

```
1. 代码块会自动关闭文件吧
2. hash.values 我觉得好像应用场景不多
3. hash.values.each 或 hash.each do |key, value| ...
4. 还可以当队列。
```

做

```
1.  x = []
    (1..16).each {x.push(rand(100))}
    puts "X = #{x.join(',')}"

    q = []
    x.each do |i|
      q.push(i)
      if q.size == 4
        puts q.join(',')
        q.clear
      end
    end
```
```ruby Day 2 Tree https://gist.github.com/tecton/a2f896e1a3eda62365db gist
2.
class Tree
  attr_accessor :children, :node_name
  
  def initialize(name_children = {})
    @node_name = name_children.keys[0]
    @children = []
    children_hash = name_children[@node_name]
    children_hash.keys.each {|c| @children.push(Tree.new({c => children_hash[c]}))}
  end
  
  def visit_all(&block)
    visit &block
    children.each {|c| c.visit_all(&block)}
  end
  
  def visit(&block)
    block.call self
  end
 
end
 
ruby_tree = Tree.new({'grandpa' => { 'dad' => {'child 1' => {}, 'child 2' => {} }, 'uncle' => {'child 3' => {}, 'child 4' => {} } } })
ruby_tree.visit_all {|node| puts node.node_name}
```

```
3.  puts "Enter filename:"
    filename = gets.chomp
    puts "Enter word:"
    target = gets.chomp

    lineNumber = 1
    File.open(filename, "r") do |infile|
      while (line = infile.gets)
        idx = line.index(target)
        if (idx != nil)
          puts "#{lineNumber}: #{line}"
        end
        lineNumber += 1
      end
    end
```

###第三天
做
```ruby Day 3 CSV Row https://gist.github.com/tecton/b2c0b7a29ff67f7dc1a9 gist
module ActsAsCsv
  
  def self.included(base)
    base.extend ClassMethods
  end
  
  module ClassMethods
    def acts_as_csv
      include InstanceMethods
    end
  end
  
  module InstanceMethods
    
    class CsvRow
      def method_missing name, *args
        @contents[@headers.index(name.to_s)]
      end
      
      attr_accessor :headers, :contents
      
      def initialize(headers, data)
        @headers = headers
        @contents = data
      end
    end
    
    def read
      @csv_contents = []
      filename = self.class.to_s.downcase + '.txt'
      file = File.new(filename)
      @headers = file.gets.chomp.split(', ')
      
      file.each do |row|
        @csv_contents << row.chomp.split(', ')
      end
    end
    
    def each(&block)
      @csv_contents.each do |row|
        block.call CsvRow.new(@headers, row)
      end
    end
    
    attr_accessor :headers, :csv_contents
    
    def initialize
      read
    end
    
  end
  
end

class RubyCsv
  include ActsAsCsv
  acts_as_csv
end

csv = RubyCsv.new
csv.each {|row| puts row.one}
```