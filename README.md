### Description

This directory contains the Twitter-Chat dataset, based on a dataset privately created by Alan Ritter, which was collected using the same procedure as in "Unsupervised Modeling of Twitter Conversations" by Ritter et al.

### Preprocessing

We used regex expressions to remove irregular punctuation marks, including double punctuation marks, round brackets and hard brackets. A few common tokens were replaced: 
- (cont) tokens were replaced with <cont> placeholder, 
- urls were replaced with <url> placeholder,
- heart icons were replaced with <heart> placeholder,
- words beginning with @ were replaced with <at> placeholder (since we have no chance of identifying the profile users are referring to),
- numbers were replaced with <number> placeholder to reduce sparsity,

Furthermore, all hashtag characters were removed as well as all non-ASCII characters.

Since the dataset was already segmented into fairly short conversations, defined by Tweets and comments to the Tweets, we did not have to do any further segmentation of the conversations. For each conversation, we appended together the utterances of each speaker to form one sequence of tokens. At the beginning of each utterance, we added one of tokens <first_speaker>, <second_speaker>, <third_speaker> or <minor_speaker> if respectively it was uttered by the first speaker (i.e. the person who posted the Tweet), the second speaker (i.e. the first person to comment on the Tweet, apart from the first speaker), the third speaker or some other speaker. Furthermore, we also appended each utterance with the end-of-utterance symbol </s>, and an end-of-dialog symbol </d> was added at the end of each utterance. These tokens should allow our model to generate a wide variety of natural dialogues.

The dataset was then tokenized with the Moses tokenizer developed by Josh Schroeder (https://github.com/moses-smt/mosesdecoder/blob/master/scripts/tokenizer/tokenizer.perl, Retrieved June, 2015). In particular, due to the differences in punctuation and casing style between movie script authors, we forced each punctuation mark to be a separate token. Finally, the dialogues were randomly partitioned into training, validation (development) and test sets.

### Statistics

Dataset		Dialogues	Tokens		Unknown tokens
Training	749060		70535057	1822755	
Validation	93633		8828201		230774
Test		93633		8793455		232183
All             936326		88156713	2285712


These statistics exclude the first </s>, which has been appended at the beginning of each dialogue for training reasons. On average, there are 94.15 tokens per dialogue. This is relatively short, compared to Movie-Scriptolog which had about 140 tokens per dialogue.

The special speech act/dialogue act tokens represent X% of all the tokens.


### Preprocessing Commands


Moses tokenization was run with the following commands:

	perl Moses/scripts/tokenizer/tokenizer.perl l=en -threads 4 < Training_Shuffled_Dataset.txt > Training_Shuffled_Dataset_Tokenized.txt
	perl Moses/scripts/tokenizer/deescape-special-chars.perl l=en -threads 4 < Training_Shuffled_Dataset_Tokenized.txt > Training_Shuffled_Dataset_Tokenized_Deescaped.txt

	sed -i 's/< \/ s\ >/<\/s>/g' Training_Shuffled_Dataset_Tokenized_Deescaped.txt
	sed -i 's/< \/ d\ >/<\/d>/g' Training_Shuffled_Dataset_Tokenized_Deescaped.txt
	sed -i 's/< first _ speaker >/<first_speaker>/g' Training_Shuffled_Dataset_Tokenized_Deescaped.txt
	sed -i 's/< second _ speaker >/<second_speaker>/g' Training_Shuffled_Dataset_Tokenized_Deescaped.txt
	sed -i 's/< third _ speaker >/<third_speaker>/g' Training_Shuffled_Dataset_Tokenized_Deescaped.txt
	sed -i 's/< minor _ speaker >/<minor_speaker>/g' Training_Shuffled_Dataset_Tokenized_Deescaped.txt
	sed -i 's/< url >/<url>/g' Training_Shuffled_Dataset_Tokenized_Deescaped.txt
	sed -i 's/< at >/<at>/g' Training_Shuffled_Dataset_Tokenized_Deescaped.txt
	sed -i 's/< cont >/<cont>/g' Training_Shuffled_Dataset_Tokenized_Deescaped.txt
	sed -i 's/< heart >/<heart>/g' Training_Shuffled_Dataset_Tokenized_Deescaped.txt
	sed -i 's/< number >/<number>/g' Training_Shuffled_Dataset_Tokenized_Deescaped.txt
	sed -i 's/ > / /g' Training_Shuffled_Dataset_Tokenized_Deescaped.txt
	sed -i 's/ = / /g' Training_Shuffled_Dataset_Tokenized_Deescaped.txt
	sed -i 's/ } / /g' Training_Shuffled_Dataset_Tokenized_Deescaped.txt
	sed -i 's/ { / /g' Training_Shuffled_Dataset_Tokenized_Deescaped.txt
	sed -i 's/ \/ / /g' Training_Shuffled_Dataset_Tokenized_Deescaped.txt
	sed -i 's/\./ \. /g' Training_Shuffled_Dataset_Tokenized_Deescaped.txt
	sed -i 's/  \+/ /g' Training_Shuffled_Dataset_Tokenized_Deescaped.txt
	sed -i 's/$/ <\/s>/' Training_Shuffled_Dataset_Tokenized_Deescaped.txt

For building pickle files:

	python scripts/convert-text2dict.py Training_Shuffled_Dataset_Tokenized_Deescaped.txt --cutoff 20000 Training
	INFO:text2dict:Total word frequency in dictionary 70535057 
	INFO:text2dict:Cutoff 20000
	INFO:text2dict:Vocab size 20004
	INFO:text2dict:Saving to Training.dialogues.pkl.
	INFO:text2dict:Saving to Training.dict.pkl.
	INFO:text2dict:Number of unknowns 1822755
	INFO:text2dict:Number of terms 70535057
	INFO:text2dict:Mean document length 94.000000
	INFO:text2dict:Writing training 749060 dialogues (0 left out)

	python scripts/convert-text2dict.py Validation_Shuffled_Dataset_Tokenized_Deescaped.txt --cutoff 20000 --dict Dataset.dict.pkl Validation
	INFO:text2dict:Vocab size 20004
	INFO:text2dict:Saving to Validation.dialogues.pkl.
	INFO:text2dict:Number of unknowns 230774
	INFO:text2dict:Number of terms 8828201
	INFO:text2dict:Mean document length 94.000000
	INFO:text2dict:Writing training 93633 dialogues (0 left out)

	python scripts/convert-text2dict.py Test_Shuffled_Dataset_Tokenized_Deescaped.txt --cutoff 20000 --dict Dataset.dict.pkl Test
	INFO:text2dict:Vocab size 20004
	INFO:text2dict:Saving to Test.dialogues.pkl.
	INFO:text2dict:Number of unknowns 232183
	INFO:text2dict:Number of terms 8793455
	INFO:text2dict:Mean document length 93.000000
	INFO:text2dict:Writing training 93633 dialogues (0 left out)



Unknown tokens make up 2.58% of the corpus.



For counting dialogues/utterances:

	grep -o -w '<\/d>' Test_Shuffled_Dataset.txt | wc -w
	grep -o -w '<\/s>' Test_Shuffled_Dataset.txt | wc -w

### MISC


	sed -i 's/\/\// /g' Training_Shuffled_Dataset.txt
	sed -i 's/ ) / /g' Training_Shuffled_Dataset.txt
	sed -i 's/ ( / /g' Training_Shuffled_Dataset.txt
	sed -i 's/ ] / /g' Training_Shuffled_Dataset.txt
	sed -i 's/ [ / /g' Training_Shuffled_Dataset.txt
	sed -i 's/\*/ \* /g' Training_Shuffled_Dataset.txt
	sed -i 's/  / /g' Training_Shuffled_Dataset.txt
	sed -i 's/  / /g' Training_Shuffled_Dataset.txt

