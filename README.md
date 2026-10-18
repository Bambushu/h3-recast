# H3 Recast

[![H3 Recast demo: one clip, seven casts](assets/poster.jpg)](assets/h3_recast_demo.mp4)

*Click the image for the 19 s demo (1080p). Every person in it is AI-generated.*

Joint character swap on MiniMax H3 + the BFS Character Swap LoRA, with optional lip repair.
Swap up to three people in a clip in one pass. The motion, the set and the clip's own audio stay.

**Try it first:** copy the files from `example_inputs/` into `ComfyUI/input/` and press Run. It is pre-set to the Distracted Boyfriend demo: all three people become retirees.

**Hard rules**
- **Number of people = everyone in the shot** (1-3), not only the ones you change. A person left out of the count gets re-dressed.
- **Scene reference = one frame of this clip**, same framing, every face visible, with everyone edited into their new look.
- **Big, obvious changes land** (age, gender, hair colour, build, painted faces). Subtle ones fall back to the clip.

**Use it** (App mode, top to bottom)
1. **Your clip**: one shot, no hard cuts, any fps (it is resampled to 24). **Duration**: a bit under its length (5.8 for a 6 s clip); the frame count rounds up to what H3 needs.
2. **Scene reference**: edit one frame of the clip (Qwen-Image, GPT Image, Photoshop). To keep someone as they are, leave them unedited and describe them as they are.
3. **Person 1-3**, left to right as they stand in the clip: their *new look* and *who they replace* ("the woman in the red dress"). These fill the author's trained caption.
4. Run. Mouth too soft? Turn **Lip repair** on, pick the person (1-3) and Run again: the swap comes from the cache and only the face pass runs (~2 min on a 5090). Lip repair strength changes re-run only the face pass too.

**If it looks wrong**
- Someone you didn't change was re-dressed: count them in Number of people and describe them as they are.
- A change didn't land: make it bigger in the scene reference. Don't give red hair to the person in the red dress (the colours mix up); swap the colours instead.
- Outfits or details drift: try a new Seed.

**What comes from where:** motion, hands, lips, accessories (sunglasses, rings, watches) and clothes come from the clip; faces and bodies from the scene reference. Want sunglasses? Put them in the scene reference. Captions: the templates live in the "People (1-3)" subgraph.

**Needs** (tested on a fresh ComfyUI 0.39.2, frontend 1.53.10; built on 0.37.2)
- **PyTorch built for CUDA 13 (cu130)**, NVIDIA driver 580+. On a cu128 PyTorch the int8 models run unoptimized and a 32 GB card runs out of memory (tested). ComfyUI warns "You need pytorch with cu130" at startup when this is wrong. Fix: `pip install torch==2.10.0+cu130 torchvision==0.25.0+cu130 torchaudio==2.10.0+cu130 --index-url https://download.pytorch.org/whl/cu130`
- **Node packs** (git clone into `custom_nodes`, then `pip install -r requirements.txt` in each). ComfyUI-Manager's "Install Missing" can't see nodes inside subgraphs, so use these links:
  - ComfyUI-KJNodes, **updated to the latest version** (an older copy lacks the MiniMax nodes): [github.com/kijai/ComfyUI-KJNodes](https://github.com/kijai/ComfyUI-KJNodes)
  - ComfyUI-VideoHelperSuite: [github.com/Kosinkadink/ComfyUI-VideoHelperSuite](https://github.com/Kosinkadink/ComfyUI-VideoHelperSuite)
  - ComfyUI-H3-FaceRefine: [github.com/Carasibana/ComfyUI-H3-FaceRefine](https://github.com/Carasibana/ComfyUI-H3-FaceRefine) (pulls ultralytics + insightface)
  - ComfyUI-H3-NativeAudioLock: not a repo of its own. Copy the folder `custom_nodes/ComfyUI-H3-NativeAudioLock` out of [github.com/Shrek3OnVH5/MiniMax-H3-NativeAudio-MusicVideo-Workflow](https://github.com/Shrek3OnVH5/MiniMax-H3-NativeAudio-MusicVideo-Workflow)
  - ComfyUI_FearnworksNodes ([github.com/fearnworks/ComfyUI_FearnworksNodes](https://github.com/fearnworks/ComfyUI_FearnworksNodes)): only for two bypassed low-VRAM nodes; without it ComfyUI shows a missing-node warning
- **Models** ([huggingface.co/Comfy-Org/MiniMax-H3](https://huggingface.co/Comfy-Org/MiniMax-H3); the Issues panel offers these as downloads): diffusion_models/minimax_h3_ref2va_pruned_int8_convrot + minimax_h3_fl2va_int8_convrot (lip repair only), text_encoders/qwen3vl_32b_minimax_h3_int8_convrot, vae/minimax_h3_video_vae_fp16 + minimax_h3_audio_vae_fp32. About 95 GB in all.
- **LoRA** into models/loras: minimax_h3_character_swap_v1.0_ref2va_r64_gl from Civitai: ["BFS - Best Face Swap" by NRDX, version 3401556](https://civitai.com/models/2027766?modelVersionId=3401556) (needs a Civitai login).
- **Lip repair**: face_yolov8m.pt from [huggingface.co/Bingsu/adetailer](https://huggingface.co/Bingsu/adetailer) into models/ultralytics/bbox. Keep it even with lip repair off: ComfyUI checks the lip-repair nodes on every run. The first lip-repair run also downloads insightface buffalo_l (~280 MB) by itself.
- Speed on an RTX 5090 32 GB: the example (1.5 MP, 6 s, 3 people) takes ~18 min; 2 MP is ~17 min per 4 s. Give it 64 GB+ system RAM.

## Files
- `assets/`: the demo video and its poster frame.
- `H3_Recast.json`: the workflow (opens in App mode; switch to Graph for the nodes).
- `example_inputs/`: the Distracted Boyfriend example (guide clip, retirees scene reference, person 2 and 3 faces for lip repair). Copy into `ComfyUI/input/`.

## Credits
- [MiniMax H3](https://huggingface.co/Comfy-Org/MiniMax-H3) (Comfy-Org repack) and ComfyUI core nodes.
- The BFS character swap LoRA and its trained `body_swap` caption by NRDX ([BFS - Best Face Swap on Civitai](https://civitai.com/models/2027766?modelVersionId=3401556)). The swap engine subgraph follows the author's workflow.
- [ComfyUI-H3-FaceRefine](https://github.com/Carasibana/ComfyUI-H3-FaceRefine) by Carasibana, ComfyUI-H3-NativeAudioLock from [Shrek3OnVH5](https://github.com/Shrek3OnVH5/MiniMax-H3-NativeAudio-MusicVideo-Workflow), [KJNodes](https://github.com/kijai/ComfyUI-KJNodes), [VideoHelperSuite](https://github.com/Kosinkadink/ComfyUI-VideoHelperSuite).
- Everyone in the example clip is AI-generated.

## License
MIT for this workflow and its example files. The models, LoRA and node packs keep their own licenses.
