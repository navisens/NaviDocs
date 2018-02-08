# Publishing NavisensPlugins for iOS

Publishing iOS plugins using cocoapods is more difficult.

The following tutorial is a very simplified version. It is recommended that you instead learn how to make your own repositories from [here](https://guides.cocoapods.org/making/index.html).

The general idea is to first generate the framework, and then push it up on a cocoapods repo.

To generate the framework, you should build your plugin for both *iphoneos* and *iphonesimulator*. Find the files by right clicking on **Products** in Xcode and using **Show in Finder**. You'll need to run a script to combine them into a universal framework, either by making a new script file in the directory, or copying those out into your own build folder. The shell script will look like this:

```bash
if [[ $# -eq 0 ]] ; then
  echo 'Must provide name of framework as argument'
  exit 1
fi
mkdir -p "Debug-iphoneuniversal" && cp -R "Debug-iphoneos/$1.framework" "Debug-iphoneuniversal/" && cp -R "Debug-iphonesimulator/$1.framework/Modules/$1.swiftmodule/." "Debug-iphoneuniversal/$1.framework/Modules/$1.swiftmodule" && lipo -create -output "Debug-iphoneuniversal/$1.framework/$1" "Debug-iphonesimulator/$1.framework/$1" "Debug-iphoneos/$1.framework/$1" && cp -R "Debug-iphoneuniversal/$1.framework" "../" && echo "Succesfully created $1.framework"
```

You should run the script by invoking `$ filename.sh PluginName`. This will generate the framework in the parent directory.

Next, you should create a new cocoapods environment. It is recommended you look online for resources to generate and validate the repository using best practices.

Otherwise, you could use a `podspec` such as the following:

```ruby
#
# Be sure to run `pod lib lint NavisensPlugins.podspec' to ensure this is a
# valid spec before submitting.
#
# Any lines starting with a # are optional, but their use is encouraged
# To learn more about a Podspec see http://guides.cocoapods.org/syntax/podspec.html
#

Pod::Spec.new do |s|
s.name             = 'NavisensPlugins'
s.version          = '1.2.0'
s.summary          = 'Provides different plugins for MotionDnaSDK.'

s.description      = <<-DESC
# This description is used to generate tags and improve search results.
#   * Think: What does it do? Why did you write it? What is the focus?
#   * Try to keep it short, snappy and to the point.
#   * Write the description between the DESC delimiters below.
#   * Finally, don't worry about the indent, CocoaPods strips it!
DESC

s.homepage         = 'https://github.com/YOUR/GITHUB/OR/REPOSITORY'
s.license          = { :type => 'BSD-3', :file => 'LICENSE' }
s.author           = { 'Author Name' => 'author.name@domain' }
s.source           = {
  :git => 'https://github.com/YOUR/GITHUB/OR/REPOSITORY.git',
  :branch => 'repositories', # recommended to use a separate branch
  :tag => s.version.to_s
}

s.ios.deployment_target = '9.1'
s.ios.vendored_frameworks = 'PATH/TO/YOUR/FRAMEWORK'

end
```

Just apply all of your own settings, and push everything up at once. Your Podfile will then be able to link to the repository, read the podspec, and determine what files it needs.
