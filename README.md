# WarpLDA: Cache Efficient Implementation of Latent Dirichlet Allocation

## Introduction

WarpLDA is a cache efficient implementation of Latent Dirichlet Allocation, which samples each token in O(1).

## Installation
Prerequisites:

	* GCC (>=4.8.5)
	* CMake (>=2.8.12)
    * git

Clone this project

	git clone https://github.com/thu-ml/warplda

Install third-party dependency

	./get_gflags.sh

Download some data, and split it as training and testing set

	cd data
	wget https://raw.githubusercontent.com/sudar/Yahoo_LDA/master/test/ydir_1k.txt
    head -n 900 ydir_1k.txt > ydir_train.txt
    tail -n 100 ydir_1k.txt > ydir_test.txt
    cd ..

Compile the project

	./build.sh
	cd release/src
	make -j

## Quick-start

Format the data

	./format -input ../../data/ydir_train.txt -prefix train
    ./format -input ../../data/ydir_test.txt -vocab_in train.vocab -test -prefix test

Train the model

	./warplda --prefix train --k 100 --niter 300

Check the result. Each line is a topic, its id, number of tokens assigned to it, and ten most frequent words with their probabilities.

	vim train.info.full.txt

Infer latent topics of some testing data.

	./warplda --prefix test --model train.model --inference -niter 40 --perplexity 10

## Data format

The data format is identical to Yahoo! LDA. The input data is a text file with a number of lines, where each line is a document. The format of each line is

    id1 id2 word1 word2 word3 ...

id1, id2 are two string document identifiers, and each word is a string, separated by white space.

## Output format

WarpLDA generates a number of files:

* `format` generates `.vocab`, where each line of it is a word in the vocabulary.
* `warplda -estimate` generates the following files
	- `.info.full.txt` is the most frequent words for each topic. Each line is a topic, with its topic it, number of tokens assigned to it, and a number of most frequent words in the format `(probability, word)`. The number of most frequent words is controlled by `-ntop`. `.info.words.txt` is a simpler version which only contains words.

  - `.model` is the model (i.e., the word-topic count matrix). The first line contains four integers
		     <size of vocabulary> <number of topics> <alpha> <beta>
	Each of the remaining lines is a row of the word-topic count matrix, represented in the libsvm sparse vector format,
	       <number of elements> index:count index:count ...
  For example, `0:2` on the first line means that the first word in the vocabulary is assigned to topic 0 for 2 times.

  - `.z.estimate` is the topic assignments of each token in the libsvm format. Each line is a document,
	        <number of tokens> <word id>:<topic id> <word id>:<topic id> ...

* `warplda -inference` generates `.z.inference`, whose format is the same as `.z.estimate`.

## Other features

* Use custom prefix for output `-prefix myprefix`
* Output perplexity every 10 iterations `-perplexity 10`
* Tune Dirichlet hyperparameters `-alpha 10 -beta 0.1`
* Use UCI machine learning repository data

    wget https://archive.ics.uci.edu/ml/machine-learning-databases/bag-of-words/vocab.nips.txt
    wget https://archive.ics.uci.edu/ml/machine-learning-databases/bag-of-words/docword.nips.txt.gz
    gunzip docword.nips.txt.gz
    ./uci-to-yahoo docword.nips.txt vocab.nips.txt -o nips.txt
    head -n 1400 nips.txt > nips_train.txt
    tail -n 100 nips.txt > nips_test.txt

## License

MIT

## Reference

Please cite WarpLDA if you find it is useful!

	@inproceedings{chen2016warplda,
	  title={WarpLDA: a Cache Efficient O(1) Algorithm for Latent Dirichlet Allocation},
	  author={Chen, Jianfei and Li, Kaiwei and Zhu, Jun and Chen, Wenguang},
	  booktitle={VLDB},
	  year={2016}
	}
