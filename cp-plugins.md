# CocoaPods Plugins

## CocoaHeads Berlin, July 2014

### Boris Bügling - @NeoNacho

![](images/CP Logo Big.png)

![20%, original, inline](images/contentful.png)

---

# Yo

![](images/OhHai.jpg)

---

# Agenda

- CocoaPods itself
- Useful plugins
- How to build your own plugin

![](images/CP Logo Big.png)

---

# CocoaPods

![](images/cp-chocolates.jpg)

---

## the ~~de-facto~~ dependency manager

![](images/Judge_Gavel.jpg)

---

![40%](images/git-submodules.png)

---

- Core
- cocoapods-downloader
- Xcodeproj
- CLAide
- CocoaPods

![](images/CP Logo Big.png)

---

# Core

```ruby 
spec = Pod::Specification.from_file('CPDColors.podspec')
puts spec.name
puts spec.version

$ ./core.rb 
CPDColors
0.1.0
```

![](images/apple-core.jpg)

---

# cocoapods-downloader

```ruby
def download_head!
	hg! %|clone #{url} #{@target_path.shellescape}| [...]
end 

def download_revision!
	hg! %|clone "#{url}" --rev '#{options[:revision]}' #{@target_path [...]
end

def download_tag!
	hg! %|clone "#{url}" --updaterev '#{options[:tag]}' #{@target_path [...]
end 
```

![](images/the_internet.jpg)

---

# Xcodeproj

```ruby
wrkspace = Xcodeproj::Workspace.new_from_xcworkspace(
	'CPDColors/Example/Demo.xcworkspace')
puts wrkspace.schemes

$ ./xcodeproj.rb 
{"Demo"=>"/Users/boris/Projects/CPDColors/Example/Demo.xcodeproj", 
 "Pods"=>"/Users/boris/Projects/CPDColors/Example/Pods/Pods.xcodeproj"}
```

![](images/Xcode.png)

---

# CLAide

```ruby
argv = CLAide::ARGV.new(['tea', '--no-milk', '--sweetner=honey'])
argv.shift_argument     # => 'tea'
argv.shift_argument     # => nil
argv.flag?('milk')      # => false
argv.flag?('milk')      # => nil
argv.option('sweetner') # => 'honey'
argv.option('sweetner') # => nil
```

![](images/Televideo925Terminal.jpg)

---

# CocoaPods

```bash
$ pod install
Analyzing dependencies
Pre-downloading: `DBCamera` from `https://github.com/[...]`
Downloading dependencies
Installing ARASCIISwizzle (1.1.0)
Installing Bolts (1.1.0)
[...]
Generating Pods project
Integrating client project
```

![](images/CP Logo Big.png)

---

# CocoaPods plugins

- Add subcommands to `pod`, the tool
- Each plugin is a Gem
- Same access as built-in commands

![](images/CP Logo Big.png)

---

# Useful plugins

![](images/tools.jpg)

---

```bash
$ pod trunk push
```

![](images/Trunk_1.jpg)

---

```bash
$ pod plugins list
```

![](images/CP Logo Big.png)

---

- Uses <https://github.com/CocoaPods/cocoapods.org/blob/master/plugins.json>

![](images/CP Logo Big.png)

---

```bash
$ pod try BBUSegmentedViewController
```

![](images/CP Logo Big.png)

---

```bash	
$ pod lib docstats
```

![](images/Melk_-_Abbey_-_Library.jpg)

---

```bash
$ pod lib testing
```

![](images/Crash-Test-Dummy-Head-038.jpg)

---

```bash
$ pod package ContentfulDeliveryAPI.podspec
```

![](images/package.jpg)

---

```bash	
$ pod roulette
```

![](images/roulette.jpg)

--- 

# How to build your own plugin

![](images/Building-His-Church.jpg)

---

# What?

- Package a Pod as a static framework
- Including dependencies
- All supported platforms
- Generate a corresponding podspec

![](images/CP Logo Big.png)

---

	$ pod package AFNetworking.podspec

![](images/CP Logo Big.png)

---

# Let's get started

![](images/start.jpg)

---

```bash
$ pod plugins create cocoapods-packager
```

![](images/CP Logo Big.png)

---

# Template

- Just a Git repo, similar to the pod template
- Reads as much from the environment as possible
- Result is installable, shippable

![](images/CP Logo Big.png)

---

# Define the command

```ruby
module Pod
	class Command
		class Package < Command
			self.summary = 'Package a podspec into a static library.'
  			self.arguments = [['NAME', :required]]

			[...]

		end
	end
end
```

---

# Parse the spec

```ruby
def spec_with_path(path)
    if !path.nil? && Pathname.new(path).exist?
      @path = path
      Specification.from_file(path)
    end
end
```

---

# run

```ruby
def run
    if @spec
      builder = SpecBuilder.new(@spec)
      newspec = builder.spec_metadata

      @spec.available_platforms.each do |platform|
        build_in_sandbox(platform)

        newspec += builder.spec_platform(platform)
      end

      newspec += builder.spec_close
      File.open(@spec.name + '.podspec', 'w') { |file| file.write(newspec) }
    else
      help! 'Unable to find a podspec with path or name.'
    end
end
```

---

# Fragment of SpecGenerator

```ruby
s.#{platform.name}.platform = :#{platform.symbolic_name}, 
	'#{platform.deployment_target}'
s.#{platform.name}.preserve_paths = '#{fwk_base}'
s.#{platform.name}.public_header_files  = 
	'#{fwk_base}/Versions/A/Headers/*.h'
s.#{platform.name}.vendored_frameworks  = '#{fwk_base}'
```

---

# Build for a specific platform

```ruby
def build_in_sandbox(platform)
    config.sandbox_root       = 'Pods'
    config.integrate_targets  = false
    config.skip_repo_update   = true

    sandbox = install_pod(platform.name)

    UI.puts 'Building framework'
    xcodebuild

    versions_path, headers_path = create_framework_tree(platform.name.to_s)
    `cp #{sandbox.public_headers.root}/#{@spec.name}/*.h #{headers_path}`

    Pathname.new(config.sandbox_root).rmtree
    Pathname.new('Podfile.lock').delete
end
```

---

# Install from generated Podfile

```ruby
def install_pod(platform_name)
    podfile = podfile_from_spec(platform_name, 
    	@spec.deployment_target(platform_name))
    sandbox = Sandbox.new(config.sandbox_root)
    installer = Installer.new(sandbox, podfile)
    installer.install!

    sandbox
end
```

---

# Generate that Podfile

```ruby
def podfile_from_spec(platform_name, deployment_target)
    name     = @spec.name
    path     = @path
    podfile  = Pod::Podfile.new do
      platform(platform_name, deployment_target)
      if path
        pod name, :podspec => path
      else
        pod name, :path => '.'
      end
    end
    podfile
end
```

---

```bash
$ rake install
```

![](images/CP Logo Big.png)

---

	$ pod package AFNetworking.podspec

![](images/CP Logo Big.png)

---

- Push your repo to GH
- Release as Ruby Gem
- Send a PR to the cocoapods.org repo

![](images/ship-it.jpg)

---

```bash
	$ git push

	$ rake release

	$ gem install cocoapods-packager
```

![](images/ship-it.jpg)

---

# 💥

![](images/TsarCannon.jpg)

---

# Future

- cocoapods-testing
- cocoapods-coverage
- cocoapods-playgrounds
- ...

![](images/Future_Sign_Exit.jpeg)

---

# Thank you!

![](images/you-yeah.gif)

---

# Links

- <http://www.objc.io/issue-6/cocoapods-under-the-hood.html>
- <http://guides.cocoapods.org>
- <https://github.com/CocoaPods/Rainforest>

![](images/CP Logo Big.png)

---

### <http://vu0.org/cp-plugins>
### @NeoNacho

![](images/CP Logo Big.png)

---

### It's a zero!

![](images/Playstation2_slim_front.jpg)
