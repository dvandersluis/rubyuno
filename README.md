# rubyuno

rubyuno is a Ruby-UNO (Universal Network Object; ['ju:nou]) bridge. UNO is used to 
construct OpenOffice.org so that you can play with the office. Rubyuno
is implemented as Ruby extension library written in C++, but the bridge 
is not so fast because value conversion and multiple API call consume time. 
Rubyuno is not well suited for task like template creation; generating ODF (Open 
Document Format) files is a better option.

rubyuno is a fork of [RUNO](https://github.com/hanya/RUNO), converted into a gem so that the extensions do not need to be compiled manually. However, it has not been tested on a wide variety of platforms. If you encounter compilation difficulties and are able to resolve them, please create a pull request so that we can get this working on as many platforms as possible!

## Installation

Add this line to your application's Gemfile:

    gem 'rubyuno'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install rubyuno

### Pre-requisites
In order for rubyuno to compile, the following must already be installed on the system:

* OpenOffice.org and OpenOffice.org SDK (tested with version 3.4).
    
    The SDK will need to be set up after installation. In order to do this, `configure.pl` within
    the SDK's installation directory (eg. /opt/openoffice.org/basis3.4/sdk) needs to be run. This
    will create an SDK directory within the current user's home directory (the script has an
    option to change the installation directory but seems to not work) with some shell scripts.
      
    The shell scripts will be named something like <code>setsdkenv_*env*.*ext*</code>, where `env`
    is your server environment and `ext` is a shell type (ie. `setsdkenv_unix.sh`).
    Please read the SDK documentation for more information.
      
* Ruby 1.9 and its headers (you may need to compile Ruby with the `--enable-shared` flag)

### Environment variables
In addition to the SDK's environment variable set up script (as mentioned above), rubyuno needs two
environment variables to function correctly.

* `LD_LIBRARY_PATH` (for Linux or UNIX) or `PATH` (for Windows)

  Used to find libraries of UNO (or OpenOffice.org). Should be setup by the `setsdkenv` script.
 
* `URE_BOOTSTRAP`
  
  Specifies fundamental(rc|.ini) file with vnd.sun.star.pathname protocol.

You may wish to update your `/etc/profile` or the like with the following to automatically set these up:

    . ~/openoffice.org3.4_sdk/$(hostname)/setsdkenv_unix.sh >/dev/null
    export URE_BOOTSTRAP="vnd.sun.star.pathname:$OFFICE_PROGRAM_PATH/fundamentalrc"

## Usage

OpenOffice must be running as a service first in order to interact with it (rubyuno uses port 2083 by default):

    soffice -headless -nologo -nofirststartwizard -accept="socket,host=localhost,port=2083;urp;StarOffice.ServiceManager"

Once it is running, you can connect to it from ruby:

    require 'rubyuno'
    ctx = Uno::Connector.bootstrap
    smgr = ctx.getServiceManager
    desktop = smgr.createInstanceWithContext("com.sun.star.frame.Desktop", ctx)
    doc = desktop.loadComponentFromURL("private:factory/swriter", "_blank", 0, [])
    doc.getText.setString("Hello from Ruby!")

### Common Errors & Resolutions

#### LoadError: cannot open shared object file

    LoadError: libuno_cppuhelpergcc3.so.3: cannot open shared object file: No such file or directory - /usr/local/lib/ruby/gems/1.9.1/gems/rubyuno-0.3.0/lib/rubyuno/rubyuno.so
    from /usr/local/lib/ruby/site_ruby/1.9.1/rubygems/custom_require.rb:36:in `require'

`$LD_LIBRARY_PATH` environment variable is not set up correctly. Ensure it contains `$OO_SDK_URE_HOME/lib`.

#### TypeError: wrong argument type Rubyuno::Com::Sun::Star::Connection::NoConnectException from <code>Uno::Connector.bootstrap</code>
`soffice` is either not running as a service or is its `-accept` option parameters do not match the arguments to `Uno::Connector.bootstrap`. Ensure the port is correct.

#### Binary URP bridge disposed during call

`$URE_BOOTSTRAP` environment variable is not set up correctly. Ensure it points to the `fundamentalrc` or `fundamental.ini` file with the `vnd.sun.star.pathname protocol` protocol.