# ![Sketch Favicon](https://raw.githubusercontent.com/woofwoofinc/sketch-favicon/master/src/assets/title.png)

[![Build Status](https://travis-ci.org/woofwoofinc/sketch-favicon.svg?branch=master)](https://travis-ci.org/woofwoofinc/sketch-favicon)
[![License](https://img.shields.io/badge/license-Apache--2.0%20OR%20MIT-blue.svg)](https://github.com/woofwoofinc/sketch-favicon#license)

[Sketch] plugin for favicon generation.

[Sketch]: https://www.sketchapp.com

Solves a problem where the author is unable to remember the ImageMagick
conversion command line and options to make `favicon.ico` files with the needed
sizes. This plugin is broadly equivalent to:

    convert icon.png -define icon:auto-resize=256,128,64,48,32,24,16 favicon.ico

Detailed documentation is provided in the [src] directories and at
[woofwoofinc.github.io/sketch-favicon].

[src]: src
[woofwoofinc.github.io/sketch-favicon]: https://woofwoofinc.github.io/sketch-favicon


Developing
----------
The project build stack uses [Node.js] and the [Yarn] package manager. Install
these on your system if they are not already available.

[Node.js]: https://nodejs.org
[Yarn]: https://yarnpkg.com

A [rkt] container build script is included in the project repository and
provides an installation which can be used to build the project also. See the
description on building and running the container in the Development Tools
Container section of the documentation for more information.

[rkt]: https://coreos.com/rkt

For macOS, [RktMachine] provides a CoreOS VM which supports developing using
the rkt container system.

[RktMachine]: https://github.com/woofwoofinc/rktmachine

Start by installing [Sketch Plugin Manager] using Yarn.

[Sketch Plugin Manager]: https://github.com/skpm/skpm

    $ yarn global add skpm

Add the other project dependencies.

    $ yarn

And build the Sketch Plugin:

    $ yarn build

It is convenient to symlink to the compiled plugin from the Sketch Plugin
directory. This means the plugin does not have to be reimported into Sketch
every time it is built. The Sketch Plugin Manager has a command for this.

    skpm link favicon.sketchplugin

This doesn't work for container development since the Sketch installation and
the skpm command are not in the same execution contexts. In this case, create
the symlink manually.

    ln -s /path/to/favicon.sketchplugin \
      ~/Library/Application\ Support/com.bohemiancoding.sketch3/Plugins/favicon.sketchplugin
      
The Sketch Plugin Manager also gives the option of disabling the plugin cache
when creating the symlink. This is convenient during development. To do this
for the container development case, use the following command:

    defaults write ~/Library/Preferences/com.bohemiancoding.sketch3.plist AlwaysReloadScript -bool YES

If you want to help extend and improve this project, then your contributions
would be greatly appreciated. Check out our [GitHub issues] for ideas or a
place to ask questions. Welcome to the team!

[GitHub issues]: https://github.com/woofwoofinc/sketch-favicon/issues

License
-------
This work is dual-licensed under the Apache License, Version 2.0 and under the
MIT Licence.

You may license this work under the Apache License, Version 2.0.

    Copyright 2017 Woof Woof, Inc. contributors

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

        http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

Alternatively, you may license this work under the MIT Licence at your option.

    Copyright (c) 2017 Woof Woof, Inc. contributors

    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:

    The above copyright notice and this permission notice shall be included in all
    copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
    SOFTWARE.

The license explainers at [Choose a License] may be helpful. They have
descriptions for both the [Apache 2.0 Licence] and [MIT Licence] conditions.

[Choose a License]: http://choosealicense.com
[Apache 2.0 Licence]: http://choosealicense.com/licenses/apache-2.0/
[MIT Licence]: http://choosealicense.com/licenses/mit/


Contributing
------------
Please note that this project is released with a [Contributor Code of Conduct].
By participating in this project you agree to abide by its terms. Instances of
abusive, harassing, or otherwise unacceptable behavior may be reported by
contacting the project team at woofwoofinc@gmail.com.

[Contributor Code of Conduct]: src/conduct.rst

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be
dual licensed as above, without any additional terms or conditions.
