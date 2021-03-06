# Filesortd [![Build Status](https://travis-ci.org/myfreeweb/filesortd.png?branch=master)](https://travis-ci.org/myfreeweb/filesortd)

A Ruby DSL for automatically sorting your files based on rules.
Like [Hazel](http://www.noodlesoft.com/hazel.php), but cross-platform, no GUI required.

## Installation

    $ gem install filesortd

If you're on OS X, also install osx-plist for things like `downloaded_from` (i.e. xattr support) to work:

    $ gem install osx-plist

## Usage

    $ filesortd start yourconfig.rb

yourconfig.rb:

```ruby
### This works everywhere:
folder "/Users/myfreeweb/Downloads" do
  # Using a single matcher
  pattern "*.mp3" do
    mv "/Users/myfreeweb/Music"

    # Do things if running on a particular OS
    os :linux do
      mv "/opt/music"
    end

    # Note that actions like `mv` update the location, so you can use them multiple times
  end

  # Using multiple matchers
  match pattern: "*.psd", pattern: %r{doc[1-2]+} do
    rm
  end
end

### This only works on Mac OS X
os :osx do
  folder "/Users/myfreeweb/Downloads" do
    downloaded_from %r{destroyallsoftware} do
      mv "/Users/myfreeweb/Movies/DAS"
      open_in :default
      # or
      open_in "MPlayerX"
    end

    kind "Ruby Source" do
      label :red
    end

    label :green do
      mv "/Users/myfreeweb/Documents"
      applescript 'tell app "Finder" to reveal theFile'
    end

    # This is the multiple matcher syntax
    match pattern: '*.mp4', downloaded_from: %r{destroyallsoftware} do
      label :gray
    end
  end
end

### You can watch multiple folders at once
folders "/Users/myfreeweb/Pictures", "/opt/pictures" do
  # Do things to any files
  any do
    label :blue
  end

  # Match by extension -- same as pattern "*.png"
  ext :png do
    pass "optipng"
    label :green
  end
end
```

### Matchers

- `any` -- any file
- `pattern(pat)` -- files that conform to `pat` (regexp or glob)
- `ext(extn)` (or `extension`) -- files that have an extension of `extn`
- `kind(knd)` -- files that have Spotlight kind of `knd`
- `label(lbl)` -- files that have Finder label of `lbl`
- `downloaded_from(url)` -- files that were downloaded from url matching `url`

Matchers can be called either by themselves

```ruby
label :gray do
  kind 'Ruby Source' do
  end
end
```

or grouped into a single statement (preferred)

```ruby
match label: :gray, kind: 'Ruby Source' do
end
```

in Ruby 1.8:

```ruby
match :label => :gray, :kind => 'Ruby Source' do
end
```

### Actions

- `contents` (or `read`) -- get the contents
- `rm` (or `remove`, `delete`, `unlink`) -- remove
- `trash` -- put to trash (OS X/Linux, Linux requires [trash-cli](https://github.com/andreafrancia/trash-cli))
- `cp` (or `copy`) -- copy
- `mv` (or `move`) -- move/rename
- `pipe(cmd)` -- start the command, pass the file to stdin, get the stdout
- `pass(cmd)` -- start the command, pass the path to the file as an argument, get the stdout
- `label(color)` -- set the OS X Finder label (:none, :orange, :red, :yellow, :blue, :purple, :green, :gray or :grey)
- `open_in(app)` -- open the file using the OS X `open` command, use :default for the default app for the file type
- `applescript(script)` -- run provided AppleScript. Use `theFile` inside it to refer to the file matched

## Contributors

- [myfreeweb](https://github.com/myfreeweb)
- [goshakkk](https://github.com/goshakkk)

## License

[MIT](https://github.com/myfreeweb/filesortd/blob/master/LICENSE.txt).
