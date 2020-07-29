(Originally written by Alvin on 2020-06-02 ~ 2020-06-03)

Documentation sources:

- https://docs.microsoft.com/en-us/windows/msix/desktop/desktop-to-uwp-manual-conversion
- https://docs.microsoft.com/en-us/uwp/schemas/appxpackage/appx-package-manifest
- https://docs.microsoft.com/en-us/previous-versions/windows/dn934795%28v%3dwin.10%29
- https://docs.microsoft.com/en-us/windows/uwp/design/style/app-icons-and-logos


Assets
---

Most of the assets starts out extracted from the package that the MSIX
Packaging Tool generated, with `fileicon.png` being the exception. Since the
assets are target-based, we need to use `makepri` to generate a `pri` file.

The assets are placed under the `Assets` dir when inside the package, so this
dir structure needs to be replicated. Also, in order to not mess up `makepri`,
the `Assets` dir needs to be placed in its own dir with no other files in it.
I chose to name it `pkg`.

I generated a `priconfig.xml` file using this command:

```
makepri.exe createconfig /cf priconfig.xml /dq en-us
```

... however I also commented out the `packaging` section.

Read more about the app icons on https://docs.microsoft.com/en-us/windows/uwp/design/style/app-icons-and-logos

TODO: On editing the assets...


Manifest
---

The manifest is constructed by referencing the official documentation and the
two manifests generated by DAC (as in Krita 4.2.8 store version) and MSIX
Packaging Tool. A huge portion is manually rewritten though.

There are several issues that couldn't be resolved:

- The property handler just doesn't work so it has been commented out.
- Explorer doesn't add the drop shadow to the thumbnail. In Win32 there is a
  registry value to change it but there is no such option in the manifest.
- Explorer doesn't load the preview image on the preview pane. In Win32 the
  image provided by the thumbnail handler is automatically used, but not for
  Appx for some reason?


Building the MSIX
---

Needs:

- Latest Windows SDK (though I've tested with 10.0.17763.0 and10.0.18362.0)
- Krita x64 NSIS installer EXE file
- 7-Zip
- The cert to sign the package with

Steps:

1. Extract the EXE installer to somewhere **outside of the source tree** using
   7-Zip (don't run the installer!)
2. Remove `$PLUGINSDIR` and `uninstall.exe.nsis` inside
3. Edit `02_gen_mapping.cmd` and point `KRITA_DIR` to it
4. Make necessary amendments to `manifest.xml`
5. Start a command prompt, then run `01_gen_resources_pri`, `02_gen_mapping`
   and `03_build_msix` in order (don't run them from Explorer or you may miss
   any error messages)
    - If you need to specify the keyfile and password used to sign the MSIX
      file, execute the following command before running the final script:
        ```
        set SIGNTOOL_SIGN_FLAGS=/f "absolute_path_to_keyfile.pfx" /p password
        ```
6. Install and test `out\out.msix`

TODO: Technically we can build the MSIX on the binary factory together with the
rest of the build (excluding the MSIX signing step)...