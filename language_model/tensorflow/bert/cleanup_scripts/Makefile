SHELL := /bin/bash

seq := $(shell echo part-{00000..00499}-of-00500)
records = $(seq)

wild_card=text/*/wiki_??

files := $(wildcard $(wild_card))

all :

# already have multi-process
cleanup_file:
	python ./cleanup_file.py --data=$(files) --output_suffix='.1'

%.2:%.1
	@echo clean $< $@
	./clean.sh $< $@

files_1 := $(wildcard text/*/wiki_??.1)

further_clean: $(files_1:.1=.2)

segmentation:
	python ./do_sentence_segmentation.py --data=$(wild_card) --input_suffix='.2' --output_suffix='.3'

seperate_set:
	python seperate_test_set.py --data=$(wild_card) --input_suffix='.3' --output_suffix='.4' --num_test_articles=10000 --test_output='./results/eval'

do_gather:
	python ./do_gather.py --data=$(wild_card) --input_suffix='.4' --block_size=26.92 --out_dir='./results'

tfrecord/%-of-00500 : results/%-of-00500
	@python create_pretraining_data.py \
		   --input_file=$< \
		   --output_file=$@ \
		   --vocab_file=./wiki/vocab.txt \
		   --do_lower_case=True \
		   --max_seq_length=512 \
		   --max_predictions_per_seq=76 \
		   --masked_lm_prob=0.15 \
	       --random_seed=12345 \
		   --dupe_factor=10

gen_tf: $(addprefix tfrecord/, $(records))

clean:
	rm -rf tfrecord/*
