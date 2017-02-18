#
# Copyright (c) 2008-2017 the Urho3D project.
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
desc 'Update Emscripten SDK in the specified branch'
task :sdk_update do
  next if timeup
  system 'git fetch origin $BRANCH && git checkout -b $BRANCH FETCH_HEAD' or abort "Failed to switch to #{ENV['BRANCH']} branch"
  system 'cd ~ && wget https://github.com/juj/emsdk/archive/master.zip && unzip master.zip && rm master.zip && cd emsdk-master && rm -rf .gitignore bin && ./emsdk install -j$NUMJOBS --build=Release sdk-$BRANCH-64bit binaryen-master-64bit' or abort 'Failed to download and install the SDK'
  next if timeup
  system 'rsync -a --delete --exclude .git --exclude binaryen --exclude clang --exclude emscripten --exclude zips ~/emsdk-master/ . && mkdir -p binaryen/master_64bit_binaryen && rsync -a --delete ~/emsdk-master/binaryen/master_64bit_binaryen/bin binaryen/master_64bit_binaryen && mkdir -p clang/fastcomp/build_${BRANCH}_64 && rsync -a --delete ~/emsdk-master/clang/fastcomp/build_${BRANCH}_64/bin clang/fastcomp/build_${BRANCH}_64 && mkdir -p emscripten/${BRANCH}_64bit_optimizer && rsync -a --delete --exclude .git ~/emsdk-master/emscripten/$BRANCH emscripten && cp ~/emsdk-master/emscripten/${BRANCH}_64bit_optimizer/optimizer emscripten/${BRANCH}_64bit_optimizer' or abort 'Failed to rsync the SDK'
  system "git config user.name '#{ENV['GIT_NAME']}' && git config user.email '#{ENV['GIT_EMAIL']}' && git remote set-url --push origin https://#{ENV['GH_TOKEN']}@github.com/urho3d/emscripten-sdk.git && git add -Af . >/dev/null 2>&1 && (git commit -q -m \"Travis CI: Emscripten SDK installation at #{Time.now.utc}.\" || true)" or abort 'Failed to commit the SDK'
  next if timeup
  if ENV['BRANCH'] == 'master'
    # This is a hack which relies on internal of emsdk
    system 'cd ~/emsdk-master/clang/fastcomp/src/tools/clang/tools && git clone --depth 1 --branch=$CLANG https://github.com/llvm-mirror/clang-tools-extra.git extra && cd ../../../../build_${BRANCH}_64 && cmake . -DLLVM_TOOL_CLANG_TOOLS_EXTRA_BUILD=1 && DESTDIR=~ cmake --build . -- -j$NUMJOBS install' or abort 'Failed to download and install extra Clang tools'
    system "cd ~/usr && git clone --depth 1 -q https://github.com/urho3d/fastcomp-clang.git && rsync -a --delete --exclude README.md --exclude .git local/ fastcomp-clang && cd fastcomp-clang && git config user.name '#{ENV['GIT_NAME']}' && git config user.email '#{ENV['GIT_EMAIL']}' && git remote set-url --push origin https://#{ENV['GH_TOKEN']}@github.com/urho3d/fastcomp-clang.git && git add -Af . >/dev/null 2>&1 && (git commit -q -m \"Travis CI: fastcomp-clang installation at #{Time.now.utc}.\" || true) && git push -q >/dev/null 2>&1" or abort 'Failed to push the fastcomp-clang'
  end
  # Delay the SDK push intentionally so it is being pushed together when the extra Clang tools are ready to be pushed
  system 'git push -q -u origin $BRANCH >/dev/null 2>&1' or abort 'Failed to push the SDK'
end

# Always call this function last in the multiple conditional check so that the checkpoint message does not being echoed unnecessarily
def timeup quiet = false, cutoff_time = 40.0
  unless File.exists?('start_time.log')
    system 'touch start_time.log split_time.log'
    return nil
  end
  current_time = Time.now
  elapsed_time = (current_time - File.atime('start_time.log')) / 60.0
  unless quiet
    lap_time = (current_time - File.atime('split_time.log')) / 60.0
    system 'touch split_time.log'
    puts "\n=== elapsed time: #{elapsed_time.to_i} minutes #{((elapsed_time - elapsed_time.to_i) * 60.0).round} seconds, lap time: #{lap_time.to_i} minutes #{((lap_time - lap_time.to_i) * 60.0).round} seconds ===\n\n" unless File.exists?('already_timeup.log'); $stdout.flush
  end
  return system('touch already_timeup.log') if elapsed_time > cutoff_time
end

# vi: set ts=2 sw=2 expandtab:
