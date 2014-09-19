# CocoaPods Plugins

## NS🍷 2014

### Boris Bügling - @NeoNacho

![](images/CP Logo Big.png)

![20%, original, inline](images/contentful.png)

---

![](images/superalloy.jpg)

-
-
-
-
-
-
-
-
-
-
-
-
-
-

## Advanced CocoaPods

---

# CocoaPods plugins

- Add subcommands to `pod`, the tool
- `post_install` hook
- Each plugin is a Gem

![](images/CP Logo Big.png)

---

## Do whatever you want, because Ruby 💎

![](images/mind_blown.gif)

---

# Useful plugins

![](images/tools.jpg)

---

```bash
$ pod plugins list
```

![](images/inception.jpg)

---

```bash	
$ pod keys set AccessToken 0xFFFFFFFF
```

![](images/keys.jpg)

---

```bash
$ pod package ContentfulDeliveryAPI.podspec
```

![](images/package.jpg)

---

```bash
$ pod lib coverage
```

![](images/coverage.jpg)

---

```bash	
$ pod roulette
```

![](images/roulette.jpg)

--- 

# How to build your own plugin

![](images/Building-His-Church.jpg)

---

```bash
$ pod plugins create cocoapods-awesome-plugin
```

![](images/CP Logo Big.png)

---

```bash
$ tree
.
├── Gemfile
├── LICENSE.txt
├── README.md
├── Rakefile
├── cocoapods_awesome_plugin.gemspec
└── lib
    ├── cocoapods_awesome_plugin.rb
    ├── cocoapods_plugin.rb
    └── pod
        └── command
            └── plugin.rb

3 directories, 8 files
```

![](images/tree.jpg)

---

```ruby
module Pod
  class Command
    class Plugin < Command
      self.summary = "Short description."

      self.arguments = [CLAide::Argument.new('NAME', true)]

      def initialize(argv)
        @name = argv.shift_argument
        super
      end

      def validate!
        super
        help! "A Pod name is required." unless @name
      end

      def run
        UI.puts "Add your implementation here"
      end
```

---

# Hooks

![](images/hook-1991.jpg)

---

```ruby
Pod::HooksManager.register(:post_install) do |options|
  require 'installer'

  UI.puts "This gets executed after installation"
end
```

---

# Thank you!

![](images/you-yeah.gif)
