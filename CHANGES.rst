=======
CHANGES
=======

---
1.1
---

This is the last build that will support installation in a pure-distutils
mode. Subsequent releases will require setuptools/distribute to install.
Python 3 installs have always had this requirement (for 2to3 install support),
but starting with the next minor release (1.2+), setuptools will be required.

Additionally, this release has made some substantial refactoring in an
attempt to modularize the backends. An attempt has been made to maintain 100%
backward-compatibility, although if your library does anything fancy with
module structure or clasess, some tweaking may be necessary. The
backward-compatible references will be removed in 2.0, so the 1.1+ releases
represent a transitional implementation which should work with both legacy
and updated module structure.

* Added a console-script 'keyring' invoking the command-line interface.
* Deprecated _ExtensionKeyring.
* Moved PasswordSetError and InitError to an `errors` module (references kept
  for backward-compatibility).
* Moved concrete backend implementations into their own modules (references
  kept for backward compatibility):

  - OSXKeychain -> backends.OS_X.Keyring
  - GnomeKeyring -> backends.Gnome.Keyring
  - SecretServiceKeyring -> backends.SecretService.Keyring
  - KDEKWallet -> backends.kwallet.Keyring
  - BasicFileKeyring -> backends.file.BaseKeyring
  - CryptedFileKeyring -> backends.file.EncryptedKeyring
  - UncryptedFileKeyring -> backends.file.PlaintextKeyring
  - Win32CryptoKeyring -> backends.Windows.EncryptedKeyring
  - WinVaultKeyring -> backends.Windows.WinVaultKeyring
  - Win32CryptoRegistry -> backends.Windows.RegistryKeyring
  - select_windows_backend -> backends.Windows.select_windows_backend
  - GoogleDocsKeyring -> backends.Google.DocsKeyring
  - Credential -> keyring.credentials.Credential
  - BaseCredential -> keyring.credentials.SimpleCredential
  - EnvironCredential -> keyring.credentials.EnvironCredential
  - GoogleEnvironCredential -> backends.Google.EnvironCredential
  - BaseKeyczarCrypter -> backends.keyczar.BaseCrypter
  - KeyczarCrypter -> backends.keyczar.Crypter
  - EnvironKeyczarCrypter -> backends.keyczar.EnvironCrypter
  - EnvironGoogleDocsKeyring -> backends.Google.KeyczarDocsKeyring
  - BasicPyfilesystemKeyring -> backends.pyfs.BasicKeyring
  - UnencryptedPyfilesystemKeyring -> backends.pyfs.PlaintextKeyring
  - EncryptedPyfilesystemKeyring -> backends.pyfs.EncryptedKeyring
  - EnvironEncryptedPyfilesystemKeyring -> backends.pyfs.KeyczarKeyring
  - MultipartKeyringWrapper -> backends.multi.MultipartKeyringWrapper

* Officially require Python 2.5 or greater (although unofficially, this
  requirement has been in place since 0.10).

---
1.0
---

This backward-incompatible release attempts to remove some cruft from the
codebase that's accumulated over the versions.

* Removed legacy file relocation support. `keyring` no longer supports loading
  configuration or file-based backends from ~. If upgrading from 0.8 or later,
  the files should already have been migrated to their new proper locations.
  If upgrading from 0.7.x or earlier, the files will have to be migrated
  manually.
* Removed CryptedFileKeyring migration support. To maintain an existing
  CryptedFileKeyring, one must first upgrade to 0.9.2 or later and access the
  keyring before upgrading to 1.0 to retain the existing keyring.
* File System backends now create files without group and world permissions.
  Fixes #67.

------
0.10.1
------

* Merged 0.9.3 to include fix for #75.

----
0.10
----

* Add support for using `Keyczar <http://www.keyczar.org/>`_ to encrypt
  keyrings. Keyczar is "an open source cryptographic toolkit designed to make
  it easier and safer for developers to use cryptography in their
  applications."
* Added support for storing keyrings on Google Docs or any other filesystem
  supported by pyfilesystem.
* Fixed issue in Gnome Keyring when unicode is passed as the service name,
  username, or password.
* Tweaked SecretService code to pass unicode to DBus, as unicode is the
  preferred format.
* Issue #71 - Fixed logic in CryptedFileKeyring.
* Unencrypted keyring file will be saved with user read/write (and not group
  or world read/write).

-----
0.9.3
-----

* Ensure migration is run when get_password is called. Fixes #75. Thanks to
  Marc Deslauriers for reporting the bug and supplying the patch.

-----
0.9.2
-----

* Keyring 0.9.1 introduced a whole different storage format for the
  CryptedFileKeyring, but this introduced some potential compatibility issues.
  This release incorporates the security updates but reverts to the INI file
  format for storage, only encrypting the passwords and leaving the service
  and usernames in plaintext. Subsequent releases may incorporate a new
  keyring to implement a whole-file encrypted version. Fixes #64.
* The CryptedFileKeyring now requires simplejson for Python 2.5 clients.

-----
0.9.1
-----

* Fix for issue where SecretServiceBackend.set_password would raise a
  UnicodeError on Python 3 or when a unicode password was provided on Python
  2.
* CryptedFileKeyring now uses PBKDF2 to derive the key from the user's
  password and a random hash. The IV is chosen randomly as well. All the
  stored passwords are encrypted at once. Any keyrings using the old format
  will be automatically converted to the new format (but will no longer be
  compatible with 0.9 and earlier). The user's password is no longer limited
  to 32 characters. PyCrypto 2.5 or greater is now required for this keyring.

---
0.9
---

* Add support for GTK 3 and secret service D-Bus. Fixes #52.
* Issue #60 - Use correct method for decoding.

-----
0.8.1
-----

* Fix regression in keyring lib on Windows XP where the LOCALAPPDATA
  environment variable is not present.

---
0.8
---

* Mac OS X keyring backend now uses subprocess calls to the `security`
  command instead of calling the API, which with the latest updates, no
  longer allows Python to invoke from a virtualenv. Fixes issue #13.
* When using file-based storage, the keyring files are no longer stored
  in the user's home directory, but are instead stored in platform-friendly
  locations (`%localappdata%\Python Keyring` on Windows and according to
  the freedesktop.org Base Dir Specification
  (`$XDG_DATA_HOME/python_keyring` or `$HOME/.local/share/python_keyring`)
  on other operating systems). This fixes #21.

*Backward Compatibility Notice*

Due to the new storage location for file-based keyrings, keyring 0.8
supports backward compatibility by automatically moving the password
files to the updated location. In general, users can upgrade to 0.8 and
continue to operate normally. Any applications that customize the storage
location or make assumptions about the storage location will need to take
this change into consideration. Additionally, after upgrading to 0.8,
it is not possible to downgrade to 0.7 without manually moving
configuration files. In 1.0, the backward compatibilty
will be removed.

-----
0.7.1
-----

* Removed non-ASCII characters from README and CHANGES docs (required by
  distutils if we're to include them in the long_description). Fixes #55.

---
0.7
---

* Python 3 is now supported. All tests now pass under Python 3.2 on
  Windows and Linux (although Linux backend support is limited). Fixes #28.
* Extension modules on Mac and Windows replaced by pure-Python ctypes
  implementations. Thanks to Jerome Laheurte.
* WinVaultKeyring now supports multiple passwords for the same service. Fixes
  #47.
* Most of the tests don't require user interaction anymore.
* Entries stored in Gnome Keyring appears now with a meaningful name if you try
  to browser your keyring (for ex. with Seahorse)
* Tests from Gnome Keyring no longer pollute the user own keyring.
* `keyring.util.escape` now accepts only unicode strings. Don't try to encode
  strings passed to it.

-----
0.6.2
-----

* fix compiling on OSX with XCode 4.0

-----
0.6.1
-----

* Gnome keyring should not be used if there is no DISPLAY or if the dbus is
  not around (https://bugs.launchpad.net/launchpadlib/+bug/752282).

---
0.6
---

* Added `keyring.http` for facilitating HTTP Auth using keyring.

* Add a utility to access the keyring from the command line.

-----
0.5.1
-----

* Remove a spurious KDE debug message when using KWallet

* Fix a bug that caused an exception if the user canceled the KWallet dialog
  (https://bitbucket.org/kang/python-keyring-lib/issue/37/user-canceling-of-kde-wallet-dialogs).

---
0.5
---

* Now using the existing Gnome and KDE python libs instead of custom C++
  code.

* Using the getpass module instead of custom code

---
0.4
---

* Fixed the setup script (some subdirs were not included in the release.)

---
0.3
---

* Fixed keyring.core when the user doesn't have a cfg, or is not
  properly configured.

* Fixed escaping issues for usernames with non-ascii characters

---
0.2
---

* Add support for Python 2.4+
  http://bitbucket.org/kang/python-keyring-lib/issue/2

* Fix the bug in KDE Kwallet extension compiling
  http://bitbucket.org/kang/python-keyring-lib/issue/3