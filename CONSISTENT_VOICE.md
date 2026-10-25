---
name: consistent-voice
description: Keep the same voice for an AI character across every video, using Claude Code + Higgsfield only. Use whenever a character talks on camera ("make a talking video of <character>", "new voice for <character>", "keep his voice the same", "voice for this persona", "swap this clip to <character>", "face swap this creator video").
---

# Consistent character voice (Claude Code + Higgsfield)

The video model does NOT keep a voice. Uploading a voice sample as an audio reference to Seedance gives a different voice every time.
So the voice is never left to the video model. It is locked in two places:

1. a fixed written **VOICE block**, pasted word for word into every prompt
2. a **cloned voice** in Higgsfield, laid over every clip afterwards with Voice Change

## One-time setup per character

1. **Write the VOICE block.** Read the character sheet and write one line, in this order:
   ethnicity + age, accent, pitch and timbre, pace and energy, personality habits (laughs, pauses), mic + room.
   Example: `VOICE (Kendrick, identical every video): Black American man, early 20s, New York City accent, clear medium-high voice, fast energetic delivery with sharp consonants, quick laughs, close phone-mic sound, dry room, no reverb.`
   Save it in `voices/<Character>.md`. Show it to the human and get a yes.
2. **Get a voice sample. No video generation for this.** Any source works: an ElevenLabs Voice Design sample, a Higgsfield text-to-speech voice, or a human recording. Have it read a varied script with a phoneme line ("That quick beige fox jumped in the air over each thin dog. Look out, I shout, for he's foiled you again, creating chaos."), in the character's energy.
   Claude cannot use ElevenLabs. If the human wants ElevenLabs, they paste the VOICE block into Voice Design, make the sample in text-to-speech, download it and give Claude the file. Otherwise make the sample with Higgsfield text-to-speech.
3. **Pick the voice.** The human listens and approves. Cut 10–30 seconds of clean speech with ffmpeg (no music, no other voices). Short and perfect beats long and mixed.
   Save the sample as `voices/<Character>_sample.mp3`. Keep it forever.
4. **Clone it.** Use the Higgsfield `create_voice` tool (Upload tab), name = the character. It costs about 40 credits, so ask first.
   Write the voice ID into `voices/<Character>.md`.

## Every video after that

Two ways. Pick by what the human gives you.

### A. New clip from a script ("make a talking video of <character> saying ...")

1. Paste the VOICE block word for word into the prompt. Never change it between videos.
2. Generate the clip in Seedance with sound ON. This voice is only a guide: it makes the lips move in time with the words.
3. Run Higgsfield `voice_change` on the clip with the character's saved voice ID. The timing stays, so the lips still match. Only the voice changes.
   Run it on whole sentences, never on stitched pieces.
4. Go to **Finish**.

### B. Swap a creator's clip ("swap this clip to <character>")

The creator's own words and timing stay. Only the face and the voice change.

1. **Make the still.** Nano Banana still of the character in the creator's first frame: same pose, same room, same framing. Give a new outfit if the human asks. Show it and get a yes.
2. **Swap the face in Wan 3.0.** The creator's clip goes in as video 1 and the still as image 1, sound OFF, 480p.
   Wan takes 15 seconds of video at most. For a longer clip, cut it at a quiet point (between sentences) into parts under 15 s and swap each part.
   Quote and ask first (about 20 credits per 13 s part at 480p).
3. **Check the swap.** Wan output can run slightly fast (about 0.8%). Compare its duration with the source part and retime it to match before joining the parts.
4. **Change the voice.** Run Higgsfield `voice_change` on the creator's ORIGINAL audio with the character's saved voice ID. Same words, same timing, so the lips still match. About 3 credits for 26 s. Quote and ask first.
5. **Put it back together.** Join the swapped parts and lay the changed voice over them. Then go to **Finish**.

## Finish (both ways): Topaz upscale to 1080p, then check

Seedance and Wan come out at 480p or 720p. Claude upscales the finished video to 1080p with the Topaz API. Never use the Higgsfield upscaler.

1. **Show the video first.** The human watches the low-res version and says yes before anything is upscaled.
2. **Key.** The Topaz API key lives in the `TOPAZ_API_KEY` environment variable (Mac: a line `export TOPAZ_API_KEY=...` in `~/.zshrc`). Never write the key in a file, a script or a message. If it is missing, ask the human to add it themselves.
3. **Upscale** with the Topaz video API (`https://api.topazlabs.com/video`, header `X-API-Key`):
   - `POST /video/` with the source (width, height, `container: mp4`, file size, duration, frameRate, frameCount) and the output (1080x1920 for 9:16, 1920x1080 for 16:9, same frameRate, `audioTransfer: Copy`, `container: mp4`), `filters: [{model: "prob-4"}]`. Show the human the cost estimate it returns and ask before going on (about 5–7 Topaz credits for 25 s).
   - `PATCH /video/<requestId>/accept` returns upload URLs. PUT the file to them (split evenly if there are several), keep each part's ETag.
   - `PATCH /video/<requestId>/complete-upload` with `{uploadResults: [{partNum, eTag}, ...]}`.
   - Check `GET /video/<requestId>/status` every 30 s until `complete`, then download `download.url`.
4. **Make it play on Mac and iPhone.** If the result is HEVC, remux it: `ffmpeg -i in.mp4 -c copy -tag:v hvc1 out.mp4`.
5. **Check before delivery** on the 1080p file: ffprobe the size, duration and audio; play it and watch the mouth on 2–3 words; transcribe it again to check the words; set loudness to -16 LUFS.

## Rules

- Never upload a voice file as a reference to the video model to "match the voice". It does not work.
- One character = one VOICE block + one cloned voice. Same ones every video.
- Higgsfield accounts only hold a few cloned voices. If a clone was deleted, re-clone it from `voices/<Character>_sample.mp3`: same sample, same voice.
- Quote and ask before every paid step (video, clone, voice change, Topaz upscale).
- Made a new character voice? Share the VOICE block and the sample with the team so everyone uses the same one.
