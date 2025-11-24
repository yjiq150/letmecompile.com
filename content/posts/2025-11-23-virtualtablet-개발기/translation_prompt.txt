# Prompt used for `.arb` translation

1. Generate Mappings

`LANG_CODE` is `th`.

Let's create a new arb translation file from source `en` to destination `{LANG_CODE}`.
For preparation, do the following first.

- Create a word mapping file for translation and write it to `en_{LANG_CODE}_mapping.txt`  (refer
  en_ko_mapping.txt for the format) in the same dir of arb.
- Don't translate proper nouns like 'VirtualTablet', 'VirtualTablet Server'
- This is a translation for Android/iOS apps that work as a tablet input device by connecting to PC
  with Bluetooth
- If you find any ambiguous words, please include those in the mapping file so that I can review.
- Don't start translation until I ask to do so.

From other LLM:

`LANG_CODE` is `th`.

Review `en_{LANG_CODE}_mapping.txt` and resolve items marked with ambiguous

- This is a translation for Android/iOS apps that work as a tablet input device by connecting to PC
  with Bluetooth
- Ask me if you need more context.


2. Start Translation

I resolved ambiguous items. re-read the file and start translation
Avoid a literal translation. Make it sound natural.

3. Review (Use Opus for better results)

`LANG_CODE` is `th`.

Compare `en` (source) and translated `{LANG_CODE}` (destination) arb file and thoroughly check
the translation correctness. refer to `en_{LANG_CODE}_mapping.txt` for more
information. and you if you have suggestions, apply first, then tell me.

