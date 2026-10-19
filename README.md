# H3 Recast

![H3 Recast loaded in ComfyUI with the example inputs and its result](assets/workflow.png)

Here's my joint character swap on MiniMax H3 + the BFS Character Swap LoRA, with optional lip repair. It swaps up to three people in a clip in one pass. The motion, the set and the clip's own audio stay.

![One clip, seven casts](assets/demo.gif)

*8 s from the demo. The full 33 s with sound is [assets/h3_recast_demo.mp4](assets/h3_recast_demo.mp4) (GitHub can't play it in the browser, hit download). Every person in it is AI-generated.*

Quick test: copy the files from `example_inputs/` into `ComfyUI/input/` and press Run. It comes pre-set to the Distracted Boyfriend demo: all three people become retirees.

## Hard rules

- Write a line for everyone in the shot (1-3 people), not only the ones you change. A person left out gets re-dressed.
- The scene reference is one frame of this clip: same framing, every face visible, everyone edited into their new look.
- Big, obvious changes land (age, gender, hair colour, build, painted faces). Subtle ones fall back to the clip.

## Use it (App mode, top to bottom)

1. Your clip: one shot, no hard cuts, any fps (it gets resampled to 24). Duration: a bit under the clip length (5.8 for a 6 s clip). The frame count rounds up to what H3 needs.
2. Scene reference: edit one frame of the clip (Qwen-Image, GPT Image, Photoshop). To keep someone as they are, leave them unedited and give them a line that describes them as they are.
3. Person 1-3, left to right as they stand in the clip, one sentence each: `the woman in the red dress becomes an old woman with long silver hair`. The word *becomes* (or *turns into*, `->`, `=`) splits who they are now from their new look. The number of people is the last line you fill in, so fill the lines top down. These fill the author's trained caption.
4. Run. Mouth too soft? Turn Lip repair on, pick the person (1-3) and Run again. The swap comes from the cache, only the face pass runs (~2 min on a 5090, ~16 min on a 16 GB card). Changing Lip repair strength also re-runs only the face pass.

## If it looks wrong

- Someone you didn't change was re-dressed: give them a line too, describing them as they are.
- A person didn't change at all: check their line has the word *becomes*.
- A change didn't land: make it bigger in the scene reference. Don't give red hair to the person in the red dress (the colours mix up); swap the colours instead.
- Outfits or details drift: try a new Seed.

What comes from where: motion, hands, lips, accessories (sunglasses, rings, watches) and clothes come from the clip. Faces and bodies come from the scene reference. Want sunglasses? Put them in the scene reference. The caption templates live in the "People (1-3)" subgraph.

## Needs (tested on a fresh ComfyUI 0.39.2, frontend 1.53.10; built on 0.37.2)

- PyTorch built for CUDA 13 (cu130), NVIDIA driver 580+. On a cu128 PyTorch the int8 models run unoptimized and a 32 GB card runs out of memory (tested). ComfyUI warns "You need pytorch with cu130" at startup when this is wrong. Fix: `pip install torch==2.10.0+cu130 torchvision==0.25.0+cu130 torchaudio==2.10.0+cu130 --index-url https://download.pytorch.org/whl/cu130`
- Node packs: git clone into `custom_nodes`, then `pip install -r requirements.txt` in each. ComfyUI-Manager's "Install Missing" can't see nodes inside subgraphs, so use these links:
  - ComfyUI-KJNodes, updated to the latest version (an older copy lacks the MiniMax nodes): [github.com/kijai/ComfyUI-KJNodes](https://github.com/kijai/ComfyUI-KJNodes)
  - ComfyUI-VideoHelperSuite: [github.com/Kosinkadink/ComfyUI-VideoHelperSuite](https://github.com/Kosinkadink/ComfyUI-VideoHelperSuite)
  - ComfyUI-H3-FaceRefine: [github.com/Carasibana/ComfyUI-H3-FaceRefine](https://github.com/Carasibana/ComfyUI-H3-FaceRefine) (pulls ultralytics + insightface)
  - ComfyUI-H3-NativeAudioLock: not a repo of its own. Copy the folder `custom_nodes/ComfyUI-H3-NativeAudioLock` out of [github.com/Shrek3OnVH5/MiniMax-H3-NativeAudio-MusicVideo-Workflow](https://github.com/Shrek3OnVH5/MiniMax-H3-NativeAudio-MusicVideo-Workflow)
  - ComfyUI_FearnworksNodes ([github.com/fearnworks/ComfyUI_FearnworksNodes](https://github.com/fearnworks/ComfyUI_FearnworksNodes)): only there for two bypassed low-VRAM nodes. Without it ComfyUI shows a missing-node warning.
- Models ([huggingface.co/Comfy-Org/MiniMax-H3](https://huggingface.co/Comfy-Org/MiniMax-H3); the Issues panel offers these as downloads): diffusion_models/minimax_h3_ref2va_pruned_int8_convrot + minimax_h3_fl2va_int8_convrot (lip repair only), text_encoders/qwen3vl_32b_minimax_h3_int8_convrot, vae/minimax_h3_video_vae_fp16 + minimax_h3_audio_vae_fp32. About 95 GB in all.
- LoRA into models/loras: minimax_h3_character_swap_v1.0_ref2va_r64_gl from Civitai: ["BFS - Best Face Swap" by NRDX, version 3401556](https://civitai.com/models/2027766?modelVersionId=3401556) (needs a Civitai login).
- Lip repair: face_yolov8m.pt from [huggingface.co/Bingsu/adetailer](https://huggingface.co/Bingsu/adetailer) into models/ultralytics/bbox. Keep the file even with lip repair off: ComfyUI checks the lip-repair nodes on every run. The first lip-repair run also downloads insightface buffalo_l (~280 MB) by itself.
- Hardware: tested on an RTX 5090 32 GB, RTX 4090 24 GB and RTX 4060 Ti 16 GB, all with 64 GB system RAM (it peaked at ~62 GB, so I would not go lower). Same output on all three. The example (1.5 MP, 6 s, 3 people) takes ~18 min on the 5090, ~26 min on the 4090 and ~92 min on the 4060 Ti; lip repair adds ~2, ~5 and ~16 min. 2 MP on the 5090 is ~17 min per 4 s.

## Files

- `assets/`: the workflow screenshot, the demo GIF and the full demo video.
- `H3_Recast.json`: the workflow. Opens in App mode, switch to Graph for the nodes.
- `example_inputs/`: the Distracted Boyfriend example (guide clip, retirees scene reference, person 2 and 3 faces for lip repair). Copy into `ComfyUI/input/`.

## Credits

- [MiniMax H3](https://huggingface.co/Comfy-Org/MiniMax-H3) (Comfy-Org repack) and ComfyUI core nodes.
- The BFS character swap LoRA and its trained `body_swap` caption by NRDX ([BFS - Best Face Swap on Civitai](https://civitai.com/models/2027766?modelVersionId=3401556)). The swap engine subgraph follows the author's workflow.
- [ComfyUI-H3-FaceRefine](https://github.com/Carasibana/ComfyUI-H3-FaceRefine) by Carasibana, ComfyUI-H3-NativeAudioLock from [Shrek3OnVH5](https://github.com/Shrek3OnVH5/MiniMax-H3-NativeAudio-MusicVideo-Workflow), [KJNodes](https://github.com/kijai/ComfyUI-KJNodes), [VideoHelperSuite](https://github.com/Kosinkadink/ComfyUI-VideoHelperSuite).
- Everyone in the example clip is AI-generated.

## License

MIT for this workflow and its example files. The models, LoRA and node packs keep their own licenses.
