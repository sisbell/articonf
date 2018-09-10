# Artifact Configuration
A program for downloading artifacts and verifying the sha hashes and pgp signatures.

### Downloading artifacts
To begin the build process, you will need to create an input file, called say urls.txt.
<pre>
https://jcenter.bintray.com/com/android/tools/annotations/24.5.0/annotations-24.5.0.pom
https://jcenter.bintray.com/com/google/guava/guava/22.0/guava-22.0.jar
https://jcenter.bintray.com/com/google/guava/guava/22.0/guava-22.0.pom
https://maven.google.com/com/android/tools/repository/26.0.1/repository-26.0.1.pom
https://maven.google.com/com/android/tools/sdk-common/26.0.1/sdk-common-26.0.1.jar
</pre>

Next run the following command
```
artc download --input urls.txt
```
This will generate files under the target directory:`./target/artc`

The **asc.tsv** file contains the fingerprint and the URL of the main artifact

<pre>
3872ED7D5904493D23D78FA2C4C8CB73B1435348 	 https://jcenter.bintray.com/com/android/tools/annotations/24.5.0/annotations-24.5.0.pom
694621A7227D8D5289699830ABE9F3126BB741C1 	 https://jcenter.bintray.com/com/google/guava/guava/22.0/guava-22.0.jar
694621A7227D8D5289699830ABE9F3126BB741C1 	 https://jcenter.bintray.com/com/google/guava/guava/22.0/guava-22.0.pom
</pre>

The **keys** directory contains any of the downloaded keys used in verifying signatures

<pre>
-rw-r--r-- 1 shane shane  2020 Sep  7 23:11 0374CF2E8DD1BDFD
-rw-r--r-- 1 shane shane  1700 Sep  7 23:08 0DA8A5EC02D11EAD
-rw-r--r-- 1 shane shane  3136 Sep  7 23:04 16AE34E5C9C3E2BB
-rw-r--r-- 1 shane shane 12289 Sep  7 23:09 205C8673DC742C7C

</pre>

Finally the **pubring.kbx** file is the keystore with the imported keys.

The **sha.tsv** file contains the sha256, followed by a gen/ver field value. 

<pre>
c3c99bf58182889fe86315e9a01473ee2c95540b9dedef898cec64554d925c54 	 gen 	 https://jcenter.bintray.com/com/android/tools/annotations/24.5.0/annotations-24.5.0.pom
1158e94c7de4da480873f0b4ab4a1da14c0d23d4b1902cc94a58a6f0f9ab579e 	 ver 	 https://jcenter.bintray.com/com/google/guava/guava/22.0/guava-22.0.jar
bfadb3b40f65dd6de1666d6b29f8bb54031396c76eeef4146cf9f28255f8bf33 	 ver 	 https://jcenter.bintray.com/com/google/guava/guava/22.0/guava-22.0.pom
a20fb26c8de5b0ff7a3069e681fcf01ebefd2f3d24b832c3af22d981d7d4376b 	 gen 	 https://maven.google.com/com/android/tools/repository/26.0.1/repository-26.0.1.pom
7e8bdca281bdcb6dad48a80854837f1c0ab46f31a0b292370e320f471f5b9cfd 	 gen 	 https://maven.google.com/com/android/tools/sdk-common/26.0.1/sdk-common-26.0.1.jar
</pre>

The _gen_ value means that the sha256 was generated by the build. There was not an associated ${url}.sha2 file to download and verify against. 

For example, this URL does not exist so its field value is _gen_

<pre>
https://maven.google.com/com/android/tools/sdk-common/26.0.1/sdk-common-26.0.1.jar.sha2
</pre>

The _ver_ field value means that the ${url}.sha2 file exists and the build verified the main artifact sha256 against this value.

### RBM
This command generates Tor RBM config files. 

<pre>
artc rbm --keyring android.gpg
</pre>

The _keyring_ option specifies the name of the keyring that you will use for the downloaded artifacts. Say that you specify the the keyring as **android.gpg**. Then the **pubring.kbx** file will be renamed to **android.gpg**. 

The following entries will be found in the **rbm/config** file. Notice that the config has a comment if the sha has not been verified. The build will only use the sha256 in if the asc file can't be used.

<pre>
    #Sha not verified from original source
  - URL:  https://maven.google.com/com/android/tools/sdk-common/26.0.1/sdk-common-26.0.1.jar
    sha256Sum: 7e8bdca281bdcb6dad48a80854837f1c0ab46f31a0b292370e320f471f5b9cfd 
  - URL:  https://jcenter.bintray.com/com/google/guava/guava/22.0/guava-22.0.jar
    sig_ext: asc
    file_gpg_id: 694621A7227D8D5289699830ABE9F3126BB741C1 
    gpg_keyring: android.gpg
  - URL:  https://jcenter.bintray.com/com/google/guava/guava/22.0/guava-22.0.pom
    sig_ext: asc
    file_gpg_id: 694621A7227D8D5289699830ABE9F3126BB741C1 
    gpg_keyring: android.gpg
  - URL:  https://jcenter.bintray.com/com/android/tools/annotations/24.5.0/annotations-24.5.0.pom
    sig_ext: asc
    file_gpg_id: 3872ED7D5904493D23D78FA2C4C8CB73B1435348 
    gpg_keyring: android.gpg
    #Sha not verified from original source
  - URL:  https://maven.google.com/com/android/tools/repository/26.0.1/repository-26.0.1.pom
    sha256Sum: a20fb26c8de5b0ff7a3069e681fcf01ebefd2f3d24b832c3af22d981d7d4376b 
</pre>

Since RBM uses flat directory structure for downloads, a script is also generated that can place the artifacts in a maven repo structure. It is found in **rbm/create_maven_repo.sh**

<pre>
# TODO: Set $M2_REPO to location of maven repository
mkdir -p $M2_REPO/com/android/tools/sdk-common/26.0.1 && cp "sdk-common-26.0.1.jar" "$_"
mkdir -p $M2_REPO/com/google/guava/guava/22.0 && cp "guava-22.0.jar" "$_"
mkdir -p $M2_REPO/com/google/guava/guava/22.0 && cp "guava-22.0.pom" "$_"
mkdir -p $M2_REPO/com/android/tools/annotations/24.5.0 && cp "annotations-24.5.0.pom" "$_"
mkdir -p $M2_REPO/com/android/tools/repository/26.0.1 && cp "repository-26.0.1.pom" "$_"
</pre>

### Package
As an alternative to generating RBM files you can package the artifacts into a maven repo format.

<pre>
artc package
</pre>

This copies the artifacts from **artifacts** directory to to the **m2** directory and then archives the m2 directory as
**maven-repo.tar.gz**.

It outputs the hash value to use in an RBM config (or other build). 

Note that for an RBM build, this archive will need to be uploaded to some location. The URL of the archive will then need to be added to the config file.

<pre>
  - URL:  https://example.com/repo/maven-repo.tar.gz
    sha256Sum: a20fb26c8de5b0ff7a3069e681fcf01ebefd2f3d24b832c3af22d981d7d4376b 
</pre>

### Additional info
Reproducible Build Manager - [https://rbm.torproject.org/](https://rbm.torproject.org/)

