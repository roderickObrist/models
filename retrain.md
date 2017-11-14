```
cd models/research/syntaxnet

# First you need to download a corpus
git clone https://github.com/UniversalDependencies/UD_English
cd UD_English

# Then you must convert from conllu to conll
wget https://raw.githubusercontent.com/dsindex/syntaxnet/master/convert.py

python convert.py < en-ud-train.conllu > en-ud-train.conll
python convert.py < en-ud-dev.conllu > en-ud-dev.conll
python convert.py < en-ud-test.conllu > en-ud-test.conll

# edit syntaxnet/context.pbtxt so that the inputs training-corpus, tuning-corpus
# and dev-corpus point to the location of your training data
cd ../
nano syntaxnet/context.pbtxt

# Train tagger
bazel-bin/syntaxnet/parser_trainer \
  --task_context=syntaxnet/context.pbtxt \
  --arg_prefix=brain_pos \
  --compute_lexicon \
  --graph_builder=greedy \
  --training_corpus=training-corpus \
  --tuning_corpus=tuning-corpus \
  --output_path=syntaxnet/models \
  --batch_size=32 \
  --decay_steps=2500 \
  --hidden_layer_sizes=128 \
  --learning_rate=0.08 \
  --momentum=0.9 \
  --seed=0 \
  --params=128-0.08-2500-0.9-0

# Prepare data for parser
for SET in training tuning test; do
 bazel-bin/syntaxnet/parser_eval \
  --task_context=syntaxnet/models/brain_pos/greedy/128-0.08-2500-0.9-0/context \
  --hidden_layer_sizes=128 \
  --batch_size=32 \
  --input=${SET}-corpus \
  --output=tagged-${SET}-corpus \
  --arg_prefix=brain_tagger \
  --graph_builder=greedy \
  --model_path=syntaxnet/models/brain_pos/greedy/128-0.08-2500-0.9-0/model
done

# Greedy parse
bazel-bin/syntaxnet/parser_trainer \
  --arg_prefix=brain_parser \
  --batch_size=32 \
  --projectivize_training_set \
  --decay_steps=2500 \
  --graph_builder=greedy \
  --hidden_layer_sizes=512,512 \
  --learning_rate=0.08 \
  --momentum=0.85 \
  --beam_size=1 \
  --output_path=syntaxnet/models \
  --task_context=syntaxnet/models/brain_pos/greedy/128-0.08-2500-0.9-0/context \
  --seed=4 \
  --training_corpus=tagged-training-corpus \
  --tuning_corpus=tagged-tuning-corpus \
  --params=512,512-0.08-2500-0.85-4
  --num_epochs=12 \
  --report_every=100 \
  --checkpoint_every=1000 \
  --logtostderr

# Prepare for structured parse
for SET in training tuning test; do
 bazel-bin/syntaxnet/parser_eval \
  --task_context=syntaxnet/models/brain_parser/greedy/512,512-0.08-2500-0.85-4/context \
  --hidden_layer_sizes=512,512 \
  --batch_size=32 \
  --beam_size=1 \
  --input=tagged-$SET-corpus \
  --output=parsed-$SET-corpus \
  --arg_prefix=brain_parser \
  --graph_builder=greedy \
  --model_path=syntaxnet/models/brain_parser/greedy/512,512-0.08-2500-0.85-4/model
done

bazel-bin/syntaxnet/parser_trainer \
  --arg_prefix=brain_parser \
  --batch_size=8 \
  --decay_steps=100 \
  --graph_builder=structured \
  --hidden_layer_sizes=200,200 \
  --learning_rate=0.02 \
  --momentum=0.9 \
  --output_path=syntaxnet/models \
  --task_context=syntaxnet/models/brain_parser/greedy/512,512-0.08-2500-0.85-4/context \
  --seed=0 \
  --training_corpus=projectivized-training-corpus \
  --tuning_corpus=tagged-tuning-corpus \
  --params=200x200-0.02-100-0.9-0 \
  --pretrained_params=models/brain_parser/greedy/512,512-0.08-2500-0.85-4/model \
  --pretrained_params_names=\
embedding_matrix_0,embedding_matrix_1,embedding_matrix_2,\
bias_0,weights_0,bias_1,weights_1

```
