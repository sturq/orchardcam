# Peri

A fork of [GrapheneOS Camera](https://github.com/GrapheneOS/Camera) that applies an
iPhone-style "look" to photos as they are saved, themed in the sturq periwinkle palette.

The camera UI, capture, and storage are GrapheneOS Camera's (MIT licensed). The
additions are a grading stage and a palette/branding pass.

## The grade

From researching Apple's pipeline and Google's openly documented HDR+: the recognizable
iPhone look is mostly **local tone mapping** (shadows lifted, highlights held back so
every region is well-exposed) plus Apple's colour rendering (warm undertone, slightly
cool shadows, restrained saturation). `AppleLook` approximates this on the GPU when a
photo is saved:

- Local tone mapping via a large-radius blur as the local average, lifting shadows and
  compressing highlights per region (a single-scale take on HDR+'s exposure fusion).
- A global S-curve for contrast.
- Warm undertone, teal-leaning shadows, restrained saturation.

It runs on Android 13+ (AGSL `RuntimeShader`); on older versions the photo is saved
ungraded. The grade is injected at one point in `capturer/ImageSaver.kt`.

## Theme

Periwinkle palette (`base #2A3042`, `primary #B9C5EE`) applied to the Material3 colour
tokens and the camera accent colours, with a periwinkle aperture launcher icon.

## Scope

The grade is applied to **saved photos**. The live preview shows the raw camera feed,
not the look: grading the live preview/video needs a CameraX effect on the camera
stream, which the current CameraX 1.6-alpha + GrapheneOS preview pipeline does not let a
custom effect attach to. That is a known limitation, not a goal of this build.

It also does not reproduce Apple's capture pipeline (multi-frame Smart HDR / Deep Fusion /
Night mode run on dedicated silicon at capture time); it renders the *look* on top of the
image the phone's camera already produced.

## Tuning

The grade is one AGSL shader plus a few constants in
`app/src/main/java/app/grapheneos/camera/AppleLook.kt`. Adjust the shadow-lift,
highlight-compression, undertone and saturation values, or replace the shader with a
sampled 3D LUT.

## Build

CI (`.github/workflows/build.yml`) builds a debug APK on every push.
