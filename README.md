# Scripts and Patches
We originally uploaded some patched APKs and full findings, but the repo was requested to be taken down. Therefore this new repo will mostly be patching scripts and instructions we used during dynamic analysis.

### Patching the Frida gadget into an APK
1. Decompile the APK using `apktool`: `apktool d -r <apk_name.apk>`
2. Download the Frida gadget corresponding to your emulator's architecture from [here](https://github.com/frida/frida/releases) <br>(the file name is `frida-gadget` followed by the version and architecture).
3. Copy the gadget into the `/lib/<your_emulator_architecture>` folder of the decompiled APK file as <br>`libfrida-gadget.so`.
4. Now, we inject the gadget loading call into the bytecode of the app, ideally before any other code is loaded, (usually the Main Activity found via the manifest file). Within that activity, inject the call via smali: 
```
const-string v0, "frida-gadget"
invoke-static {v0}, Ljava/lang/System;->loadLibrary(Ljava/lang/String;)V
```
5. Check if internet permissions are already in the manifest, if not add it: 
```
<uses-permission android:name="android.permission.INTERNET" />
```
6. Repackage the application
```
apktool b <target_folder> -o <output_name.apk>
```
7. Sign and zipalign the APK
```
keytool -genkey -v -keystore custom.keystore -alias mykeyaliasname -keyalg RSA -keysize 2048 -validity 10000

jarsigner -sigalg SHA1withRSA -digestalg SHA1 -keystore mycustom.keystore -storepass mystorepass <output_name.apk> mykeyaliasname

jarsigner -verify <output_name.apk>

zipalign 4 <output_name.apk> <output_name_aligned.apk>
```

### Patching Frida into APK automatically with Objection
1. Install `frida-tools` using `pip install frida-tools`
2. Install `objection` using `pip install objection` 
3. To patch an APK file in the current directory, run `objection patchapk --source <apk_name.apk>` 
4. Objection will try to automatically determine the architecture of your emulated device, but if it fails you can manually specify it using the `--architecture` flag.

### Disabling SSL Pinning using Objection
1. Open up the patched application
2. Use `objection explore` to hook with the Frida gadget
3. Disable SSL Pinning with `android sslpinning disable`