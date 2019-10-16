#Install ImageMagick for Windows for PHP 7.1 MSVC14 (Visual C++ 2015)
This guide is valid for PHP 7.1 or lower as the `.dll` files for `Imagick` extension are only available for versions prior to PHP 7.2.
https://ourcodeworld.com/articles/read/349/how-to-install-and-enable-the-imagick-extension-in-xampp-for-windows
1. Download and install ImageMagick for Windows
https://www.imagemagick.org/script/download.php#windows
```
magick -version
```
2. Download Imagick for PHP
https://pecl.php.net/package/imagick
https://pecl.php.net/package/imagick/3.4.3/windows
https://windows.php.net/downloads/pecl/releases/imagick/3.4.3/php_imagick-3.4.3-7.1-ts-vc14-x64.zip
`php_imagick-<version>-<thread-safe-or-not>-<php-compiled-version>-<architecture>.zip`
`php_imagick.dll`
```
extension=php_imagick.dll
```
3. Download required Imagick binaries
https://windows.php.net/downloads/pecl/deps/
open the bin folder and copy all the .dll files (except ImageMagickObject.dll) that would be about 146 files (with prefixes CORE_* and IM_MOD_*) 

##In case of console error
```
Warning: PHP Startup: Unable to load dynamic library 'e:/wamp/bin/php/php7.1.22/ext/php_imagick.dll' - The specified module could not be found.
```
You will need to add the bin directory of Apache (C:\xampp\apache\bin) to the PATH environment variable of Windows and the problem will be solved.

