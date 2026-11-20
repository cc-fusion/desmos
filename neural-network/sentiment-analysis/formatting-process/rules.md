Patch the attached file by creating an HTML file with hardcoded patches. The user should be able to upload the file and download the newly updated CSV file
Rules:
- Convert to CSV format
- Everything must stay lowercase
- Convert slang and very informal sentences to be slightly more formal. Correct typos
- Remove all punctuation and non-alphabetical characters. Contractions (don't, can't) are ok. Convert numbers and symbols to words (eg. 2 -> two, $2 -> two dollars)
- Remove all text artifacts (metadata, speaker ID, rating scores, etc...)
- Remove rows that do not form real sentences (after punctuation removed)
- Create a new column with a sentiment label. Label as either positive, negative, or neutral
- Remove rows that give a mixed sentiment
- Remove rows containing sarcasm
- Merge rows that appear to contain a single sentence split across multiple rows
- Remove duplicate rows
- Remove blank rows
- Remove icons like :) or :p
