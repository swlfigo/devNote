# XCode Product文件夹消失



1. Right click your `xcodeproj` file and choose "Show Package Contents"
2. Make a copy of the `pbxproj` file in case something goes wrong
3. Open the `pbxproj` for editing, search for productRefGroup and delete that line.
4. Save re-open Xcode project
5. Products directory shows back up!



## Reference

https://stackoverflow.com/questions/70635415/xcode-13-missing-products-folder-when-creating-a-framework

