Installing Synthea on a Windows system that hasn't had any Ruby on it before took a little more effort. These are all the steps I had to do on a Windows 7 X64 machine. An alternative is to use a [[Docker]] container.

Note: the 'cd', 'git' and other commands are to be run from a command line window (cmd.exe).

1. Download Ruby 2.3.1 (X64) from RubyInstaller.org (direct [link](http://dl.bintray.com/oneclick/rubyinstaller/rubyinstaller-2.3.1-x64.exe))
    1. run the installer and install it to C:\Programs\Ruby23-x64\
2. Download the DevKit for Ruby 2.0 and above, for x64 from RubyInstaller.org (direct [link](http://dl.bintray.com/oneclick/rubyinstaller/DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe))
    1. run the installer: it wil auto-unzip,  
        unzip it to C:\Programs\Ruby23-x64\DevKit
    2. register the DevKit in your path with devkitvars.bat / .ps1, included in this directory  
        if you skip this, you wil get in trouble with the installation of bson, a part of step 5e.
3. Log off & on again to reload the changed environment variables.
4. Install rubygems-update 2.6.7, primarily to get an updated certificate
    1. otherwise you will encounter:

        ```SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed```
    
    2. we follow [these instructions](http://guides.rubygems.org/ssl-certificate-update/):
        1. Download the [update](https://rubygems.org/downloads/rubygems-update-2.6.7.gem)
        2. cd c:\programs\ruby (or anywhere, gem is now in your path).
        3. gem install --local <wherever you downloaded it>\rubygems-update-2.6.7
        4. update_rubygems --no-ri --no-rdoc

5. Now we can follow the instructions on the [Synthea Git page](https://github.com/synthetichealth/synthea)
    1. cd c:\git
    2. git clone https://github.com/synthetichealth/synthea.git
    3. cd synthea
    4. gem install bundler
    5. bundle install
6. Check your hosts file:
    1. %SystemRoot%\System32\drivers\etc\hosts (you can open it in Notepad.exe)
    2. make sure the line '127.0.0.1 localhost' is present and uncommented.
    3. otherwise the Ruby MongoDB driver Moped / Mongoid won't find your Mongo service.
