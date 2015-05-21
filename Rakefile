#
# Copyright (c) 2008-2015 the Urho3D project.
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
# THE SOFTWARE.
#

task default: %w[sdk_update]

# Usage: NOT intended to be used manually
desc 'Update Emscripten SDK in the master branch'
task :sdk_update do
  system 'git fetch origin master && git checkout -b master FETCH_HEAD' or abort 'Failed to switch to master branch'
  system "wget https://s3.amazonaws.com/mozilla-games/emscripten/releases/emsdk-portable.tar.gz && tar xfz emsdk-portable.tar.gz --strip-components=1 && rm emsdk-portable.tar.gz && ./emsdk update >/dev/null && sed -i.bak 's/-xvf/-xf/g' emsdk && ./emsdk install -j$NUMJOBS --build=Release sdk-incoming-64bit" or abort 'Failed to download and install the SDK'
  system 'mv clang/fastcomp/build_incoming_64/bin clang-keep && mv emscripten/incoming_64bit_optimizer/optimizer optimizer-keep && rm -rf clang/fastcomp emscripten/incoming_64bit_optimizer emscripten/incoming/.git emsdk.bak && mkdir -p clang/fastcomp/build_incoming_64 && mv clang-keep clang/fastcomp/build_incoming_64/bin && mkdir -p emscripten/incoming_64bit_optimizer && mv optimizer-keep emscripten/incoming_64bit_optimizer/optimizer' or abort 'Failed to remove source and build scripts'
  system "git config user.name '#{ENV['GIT_NAME']}' && git config user.email '#{ENV['GIT_EMAIL']}' && git remote set-url --push origin https://#{ENV['GH_TOKEN']}@github.com/urho3d/emscripten-sdk.git && git add -Af . >/dev/null 2>&1 && ( git commit -q -m \"Travis CI: Emscripten SDK installation at #{Time.now.utc}.\" || true) && git push -q -u origin master >/dev/null 2>&1" or abort 'Failed to push new SDK installation'
end

# vi: set ts=2 sw=2 expandtab:
