# Solution

This issue has been resolved and the example adjusted to work with daemons. The problem was that the example binds itself to the stdout (writing any received data to it).
When running in a daemon the stdout is a file instead, causing a non-file check in bootstrap to fail.

# Reproduction

To run the reproduction follow these steps:
- Build using Xcode (Product > Build)
- Find the executable in the build folder (Product > Show Build Folder in Finder)
- Move the executable to `/Library/PrivilegedHelperTools`
- Create a daemon configuration in `/Library/LaunchDaemons/nio-ssh-daemon.plist`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
	<dict>
		<key>Label</key>
		<string>nio-ssh-daemon</string>
		<key>ProgramArguments</key>
                 <array>
                 	<string>/Library/PrivilegedHelperTools/nio-ssh-daemon</string>
	    		<string>username:password@host</string>
			<string>ls -la</string>
                 </array>
		<key>KeepAlive</key>
		<true/>
		<key>ProcessType</key>
		<string>Interactive</string>
		<key>StandardOutPath</key>
		<string>/Library/Logs/nio-ssh-daemon.out.log</string>
		<key>StandardErrorPath</key>
		<string>/Library/Logs/nio-ssh-daemon.err.log</string>
	</dict>
</plist>
```
making sure to adjust the program arguments to include an host with username and password.
- Load the daemon using 
```
sudo launchctl load nio-ssh-daemon.plist
```
- When opening `Console.app`, navigating to `Log Reports` and opening `nio-ssh-daemon.out.log` the logged error will be shown:
```
Creating bootstrap
Connecting channel
Creating child channel
Waiting for connection to close
Error in pipeline: operationUnsupported
An error occurred: commandExecFailed
```

If the executable is run manually without a daemon it will work correctly:
```
./nio.ssh-daemon username:password@host
```

The reproduction is a copy of the example in the repository (https://github.com/apple/swift-nio-ssh/tree/main/Sources/NIOSSHClient) with slight modifications to log errors instead of using `try!`.
