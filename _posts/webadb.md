
Checkout to 3a14c162d640e6588b01244ba46d60e28b8be9ba Tue Sep 12 19:30:51 2023 +0800

```
diff --git a/apps/demo/src/components/scrcpy/state.tsx b/apps/demo/src/components/scrcpy/state.tsx
index 5aa736ea..dbfa0baa 100644
--- a/apps/demo/src/components/scrcpy/state.tsx
+++ b/apps/demo/src/components/scrcpy/state.tsx
@@ -243,10 +243,10 @@ export class ScrcpyPageState {
                     decoderDefinition.Constructor.capabilities[
                         SETTING_STATE.settings.videoCodec!
                     ];
-                if (capability) {
-                    videoCodecOptions.value.profile = capability.maxProfile;
-                    videoCodecOptions.value.level = capability.maxLevel;
-                }
+                // if (capability) {
+                //     videoCodecOptions.value.profile = capability.maxProfile;
+                //     videoCodecOptions.value.level = capability.maxLevel;
+                // }
             }
 
             // Disabled due to https://github.com/Genymobile/scrcpy/issues/2841
diff --git a/apps/demo/src/pages/install.tsx b/apps/demo/src/pages/install.tsx
index 8a67ec4c..2c0b4c74 100644
--- a/apps/demo/src/pages/install.tsx
+++ b/apps/demo/src/pages/install.tsx
@@ -8,7 +8,7 @@ import {
     PackageManager,
     PackageManagerInstallOptions,
 } from "@yume-chan/android-bin";
-import { WrapConsumableStream, WritableStream } from "@yume-chan/stream-extra";
+import { WrapConsumableStream } from "@yume-chan/stream-extra";
 import { action, makeAutoObservable, observable, runInAction } from "mobx";
 import { observer } from "mobx-react-lite";
 import { NextPage } from "next";
@@ -110,13 +110,13 @@ class InstallPageState {
         );
 
         const elapsed = Date.now() - start;
-        await log.pipeTo(
-            new WritableStream({
-                write: action((chunk) => {
-                    this.log += chunk;
-                }),
-            })
-        );
+        // await log1.pipeTo(
+        //     new WritableStream({
+        //         write: action((chunk) => {
+        //             this.log += chunk;
+        //         }),
+        //     })
+        // );
 
         const transferRate = (
             file.size /

```
