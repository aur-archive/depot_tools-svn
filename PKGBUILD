# Maintainer: rway <rway07@gmail.com>
# Contributor: wabi <aschrafl@jetnet.ch>
# Contributor: Alexander RÃ¸dseth <rodseth@gmail.com>
# Contributor: Andreas Schrafl <aschrafl@gmail.com>
# Contributor: piojo <aur@zwell.net>
# Contributor: hack.augusto <hack.augusto@gmail.com>

pkgname=depot_tools-svn
pkgver=294236
pkgrel=1
pkgdesc='Build tools for working with Chromium development, include gclient'
arch=('any')
url='http://dev.chromium.org/developers/how-tos/install-depot-tools'
source=('depot_tools.sh' 'repo_fix.sh')
license=('Custom')
depends=('python2' 'python2-colorama' 'python2-pylint')
makedepends=('git' 'scons' 'setconf')
provides=('depot_tools' 'gclient')
conflicts=('gclient-svn')
options=('!strip')
md5sums=('bfbabdaef2cb6599e4dc385dc16541bc'
	 'fb0c546a078c312aa64c1f2a31599557')
install="depot_tools.install"
_svntrunk="http://src.chromium.org/svn/trunk/tools/"
_svnmod="depot_tools"

package()
{
	# Creating directories
	install -d "$pkgdir/opt"
	cd "$pkgdir/opt"

	# Fetching tools from svn repository
	msg "Obtaining tools from Subversion Repository"
	svn co $_svntrunk/$_svnmod -r $pkgver $pkgname
	msg "SVN Checkout (maybe) done"

	cd $pkgname

	# This tools work with python2, but ArchLinux default is python3. Fix it.
	# pylint is in extra, ninja is an executable and it does not need any change.
	# gclient.py require a fix for work correctly with python2-colorama
	# Another way is make default python2, but I don't think is a good idea!
	# Fixing python scripts.
	msg "Patching script for python2 usage..."
	for script in *.py
	do
		sed -r 's/#!(.*)python.*/#!\1python2/' $script > "$script.temp"
		install -Dm755 "$script.temp"  $script
	done;

	# Fixing gclient.py
	script=gclient.py
	sed -r  -e 's/#!(.*)python.*/#!\1python2/' \
		-e 's/from third_party import colorama/import colorama/' \
		-e 's/from third_party.colorama import Fore/from colorama import Fore/' \
		-e "s/% new_url/% new_url)/g" \
		-e "s/print 'Overwriting/print ('Overwriting/g" \
		$script > "$script.temp"
	install -Dm755 "$script.temp" $script


	# Fixing gclient
	script=gclient
	sed -r -e 's/exec python/exec python2/g' $script > "$script.temp"
	install -Dm755 "$script.temp" $script

	# Fixing repo
	script=repo
	sed -r -e 's/env python/env python2/' $script > "$script.temp"
	install -Dm755 "$script.temp" $script

	# Fixing scripts in root folder
	for script in {apply_issue,drover,gcl,git-cl,git-gs,git-try,hammer,weekly,wtf,update_depot_tools}
	do
		sed -r  -e 's/exec python/exec python2/' \
			-e 's/#!(.*)python.*/#!\1python2/' \
			$script > "$script.temp"
		install -Dm755 "$script.temp" $script
	done;

	# Fixing scripts in other folders
	# Is it safe remove those folders?? I don't now, further tests required. For now fix it and include all tools.
	script=git-cl-upload-hook
	sed -r -e 's/python/python2/' $script > "$script.temp"
	install -Dm755 "$script.temp" $script

	for script in {git_utils/git-tree-prune,support/chromite_wrapper,tests/sample_pre_commit_hook,third_party/pylint/epylint.py}
	do
		sed -r -e 's/env python/env python2/' \
			$script > "$script.temp"
		install -Dm755 "$script.temp" $script
	done;

	for folder in {testing_support,tests,third_party,win_toolchain}
	do
		cd $folder
		for script in *.py
		do
			sed -r -e 's/env python/env python2/' \
				$script > "$script.temp"
			install -Dm755 "$script.temp" $script
		done;
		cd ..
	done;

	# Export PATH
	install -Dm755 "$srcdir/depot_tools.sh" "$pkgdir/etc/profile.d/depot_tools.sh"

	# Install repo_fix.sh script
	install -Dm 755 "$srcdir/repo_fix.sh" "$pkgdir/opt/$pkgname"

	# Install License
	install -Dm644 "$pkgdir/opt/$pkgname/LICENSE" "$pkgdir/usr/share/licenses/$pkgname/LICENSE"

	# Cleaning temporary files
	msg "Cleaning temp files"
	cd "$pkgdir/opt/$pkgname"
	for file in *.temp
	do
		rm $file
	done;

	# Removing .svn folder
	# TODO: Check if update_depot_tools command works.
	rm -rf $pkgdir/opt/$pkgname/.svn

	# TODO: Check depot_tools upgrade.
}
