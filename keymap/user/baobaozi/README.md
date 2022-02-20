
# Better instructions

Because the instructions aren't great.

No combos as yet: https://github.com/kmonad/kmonad/issues/157

## macOS

1. So if you want to set it up like QMK, you would fork off the kmonad repo
1. Then you'd clone your own one. You need to recursively clone (it has repos within it that you want too, and if you didn't recursively clone, they wouldn't be there)
  `git clone --recursive https://github.com/[git-user]/kmonad.git`
1. It compiles via `stack` and if you don't have it, you need to get it. Do you have homebrew set up? it's a package manager for macOS, based off the linuxbrew
1. https://brew.sh/ install it via terminal with the command: `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`
1. Then you should just need to install stack via `brew install haskell-stack`
1. You may need to install ghcup from Haskell (https://www.haskell.org/ghcup/) just so you can install ghc ... an easier way to do this may just be to `brew install python3`
1. Then, within that kmonad repo, you need to navigate into: `kmonad/c_src/mac/Karabiner-DriverKit-VirtualHIDDevice/dist` and install `Karabiner-DriverKit-VirtualHIDDevice-1.15.0.pkg`
1. Then activate this thing with `/Applications/.Karabiner-VirtualHIDDevice-Manager.app/Contents/MacOS/Karabiner-VirtualHIDDevice-Manager activate`
   - If you get an error that says:
      ```
      activation of org.pqrs.Karabiner-DriverKit-VirtualHIDDevice is requested
      request of org.pqrs.Karabiner-DriverKit-VirtualHIDDevice is canceled because newer version (1.6.0) is already installed
      request of org.pqrs.Karabiner-DriverKit-VirtualHIDDevice is failed with error: The operation couldn’t be completed. (OSSystemExtensionErrorDomain error 11.)
      ```
      That's because you have a newer version of Karabiner (v14+) installed. You have to uninstall it, and instead of `activate` in the command above, use `forceActivate`
1. Double-check that it's the right version (should be 1.15.0) with `defaults read /Applications/.Karabiner-VirtualHIDDevice-Manager.app/Contents/Info.plist CFBundleVersion`
   And no, version 1.22.0 doesn’t work.
1. **IF YOU ARE ON AN APPLE M1 CHIP**, you have to `brew install ghc`, and then update the stack.yaml to include two new lines: `system-ghc: true` and `install-ghc: false` (read: https://github.com/kmonad/kmonad/issues/334#issuecomment-899515001)
1. Now you can install the kmonad executable: `stack install --flag kmonad:dext --extra-include-dirs=c_src/mac/Karabiner-DriverKit-VirtualHIDDevice/include/pqrs/karabiner/driverkit:c_src/mac/Karabiner-DriverKit-VirtualHIDDevice/src/Client/vendor/include`
1. Then you have to add `~/.local/bin` to your PATH so your system knows where to find it (https://gist.github.com/nex3/c395b2f8fd4b02068be37c961301caa7 if you need help doing that)
1. And you will need to give it sufficient permission to remap your keys, by adding it to System Preferences > Security & Privacy > Privacy > Input Monitoring (you can try running it first and getting a denied error, then the popup can take you there and you can check the box to allow. You'll need to restart your terminal. Probably have to add permissions for Terminal and kmonad and maybe some of the Karabiner dext shit too, maybe)
1. And to run it, you have to use sudo: `sudo kmonad keymap/user/baobaozi/colemak-baobaozi-ansi.kbd`
1. It'll ask for your password to allow sudo, then, if it works, it'll say `connected`. If it doesn't work, there'll be a tonne of errors

So … that's it. Better than their docs by a long shot.

### macOS auto-run on log in (only works after password entered after logged in)

~~Follow this for auto-load https://github.com/kmonad/kmonad/issues/105#issuecomment-700554035 but it may not work and you may bork your computer.~~ Do this at your own risk. I activated my company’s lockdown software because it believed I was “doing something dodgy”, had to go into office to get it unlocked with a glaring, “what were you trying to do?” Also, it didn’t work anyway.

What follows under this I’m using now, which has the benefit of being pretty easy to do, and actually works.

1. Create a new Automator Application, drag in `Run Applescript` and add the following:
   ```shell
   on run {input, parameters}

       tell application "Terminal"
       activate
       do script with command "sudo /Users/[username]/.local/bin/kmonad /Users/[username]/repos/personal/kmonad/keymap/user/baobaozi/colemak-baobaozi-ansi.kbd"
       end tell

   end run
   ```
1. Save it to `/Applications/` call it something like `kmonad-autorun.app`
1. Place your saved `.app` as a login item in System Preferences > Users & Groups > [your user] > Login Items
1. When you run the app or log in, a Terminal window will open and ask for your password to run the command in sudo. Enter it.
1. On first run, it’ll get a Permissions denied, go where it tells you in System Preferences to allow Terminal permission to do the thing (System Preferences > Security & Privacy > Privacy > Input Monitoring)—you can double click your `.app` specifically for the purpose of getting to this screen for adding the correct Input Monitoring permissions in.
1. If everything’s right, it’ll show the familiar `connected` text. Since the kext is already loaded, we don’t need some shell script to load the kext before running the script—we’re running kmonad only after log in and the kext has already loaded.

That’s it.

## Linux

The instructions on kmonad are much better for linux, because it was first created for it. Just a couple of parts I felt needed clarification.

1. You need gcc and stack: `pamac install gcc stack`
1. `stack install` will download GHC and compile kmonad to `~/.local/bin`
1. Then you have to add `~/.local/bin` to your PATH so your system knows where to find it (https://gist.github.com/nex3/c395b2f8fd4b02068be37c961301caa7 if you need help doing that)
1. You need some `uinput` group, so to create one: `sudo groupadd uinput`
1. Then you need to put yourself in it, so:
   ```shell
   sudo usermod -aG input [username]
   sudo usermod -aG uinput [username]
   ```
1. You can check the groups are there and you’re in them by running `groups`
1. In `/lib/udev/rules.d` sudo create a `99-kmonad.rules` (the number doesn’t seem to matter) that has:
   ```shell
   KERNEL=="uinput", MODE="0660", GROUP="uinput", OPTIONS+="static_node=uinput"
   ```
1. Then log out and back in, and check that the `uinput` drivers are loaded with this command (you should not get an error from this): `sudo modprobe uinput` (and you probably should check this every time before running your keymap)
1. Now you can run your keymap, but because of the enter key releasing, if you simply run it you will have a crazy time locking your keyboard and maybe trackpad if it’s a laptop. So instead you have to run:
   ```shell
   kmonad path/to/the/keymap.kbd -w 1000
   ```
   So that there’s enough time after the release of your enter key, to then activate.

### Linux autorun on boot (already works on log in screen!)
1. Navigate to `/etc/systemd/system` and `sudo vim kmonad.service`
1. Paste in and then edit the necessary command at `ExecStart`:
   ```shell
   [Unit]
   Description=kmonad keyboard config

   [Service]
   Restart=always
   RestartSec=3
   ExecStart=/home/[username]/.local/bin/kmonad /home/[username]/repos/personal/kmonad/keymap/user/baobaozi/colemak-baobaozi-ansi-nix.kbd
   Nice=-20

   [Install]
   WantedBy=default.target
   ```
1. Save and exit vim
1. Update permissions to make it executable: `sudo chmod 755 kmonad.service`
1. Then enable the service: `sudo systemctl enable kmonad.service`

### For your keymap

1. To find your keyboard, use the keyboard devices listed
under `/dev/input/by-id`.
1. **Not** required to have `"/bin/sleep 1"` after the output board name in the keymap (eg. `output (uinput-sink "baoboard" "/bin/sleep 1")`)

## Windows 11

Windows isn't able to do one-shots? If you set the base OS keyboard layout to English International, it seems to screw up with the `'` key, requiring a second press before it will then output 2 primes? If you press another character after, it somehow acts as if it were an alt? Like, produces `á`? Make sure you use US Qwerty.

The release on github is mega old. To compile a new exe, you need PowerShell, scoop to install stack and git. But then you realise that the latest master is even buggier than the 0.41.0 windows exe release that’s mega old. But maybe after the keycode-refactor it’ll work better, and we’ll be able to use one-shots.

But the process is much easier (or so it seems.) Download the executable from Releases, then create a shortcut to it and add to the target (after the second `"`, so Target would say something like `"C:\Program Files\kmonad\kmonad.exe" C:\Users\me\repos\personal\kmonad\keymap\user\baobaozi\colemak-baobaozi-ansi-win.kbd`) so that it’ll run kmonad against your keymap. Then you can simply press Win+R to run, and type `shell:startup` (or `shell:common startup` for all users) and then drop that shortcut into that directory.