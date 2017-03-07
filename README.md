# Why is this patch needed?
`libusb` implemented hotplug events on version `1.0.16` on some platforms (`Mac OS X` included). The problem is that in Mac's implementation, every time we want to list all available devices, open a given device, etc. there is a 5s delay induced (where `libusb` simply waits for hotplug events doing nothing).

This is annoying and slow, so I made this patch to disable hotplug polling, thus **getting rid of these unnecessary delays**.


# Editing libusb
The only change required to disable hotplug polling is in file `libusb/os/darwin_usb.c` (`libusb`'s GitHub repo is located at https://github.com/libusb/libusb). We just need to find the definition of `const struct usbi_os_backend darwin_backend` and set the field `.hotplug_poll` to `NULL`  (as of version `1.0.21` this is at line `2059`). That is, we need to change line `2059` from:
```c
/* Line 2059 */ .hotplug_poll = darwin_hotplug_poll,
```
to:
```c
/* Line 2059 */ .hotplug_poll = NULL,
```

Finally, we could `git diff >> libusb.diff` to save our changes to a `diff` file.

# Editing Homebrew's libusb Formula
Now that we've made the changes, we just need to apply the patch to `Homebrew`'s `libusb` Formula. We can edit the formula by typing:
```sh
brew edit libusb
```

Simply add these lines before the `bottle do` block:
```ruby
patch do
  url "https://github.com/CarlosRDomin/libusb_fixHotplug_MacOsX/libusb.diff"
  sha256 "261d5139dd585010b39633d8c94bb09700e355b5c6c6b5e46b17eacec9e9b6fb"
end
```

Note that you can easily compute the sha256 of the patch by running:
```sh
shasum -a 256 libusb.diff
```

Last, `install` (or `reinstall`) `libusb` **from source** (so the patch gets applied, the regular *bottled* version won't contain our patch!):
```sh
brew install -s libusb
```
