## Bert  中文文本分类


## 运行环境
- python3.6(使用python2的话，注意修改run_classifier_0214.py中SelfProcessor的encoding)
- tensorflow1.12(Bert官方支持的版本均可)

## 数据说明
- 运行时数据所在目录为/Users/luyao/Desktop/bert_learn/MAYI，在这里为了方便起见我把数据一起上传了
- train.tsv、val.tsv、test.tsv格式相同，各列分别为"index query1 query2 label"，"\t"分隔。摘取了网络上蚂蚁金服的公开数据集一部分，是判断文本相似性的数据

## 程序说明
>如下的目录都是我自己学习时的目录，大家根据自己的实际情况进行修改
- 首先要下载**[`BERT-Base, Chinese`](https://storage.googleapis.com/bert_models/2018_11_03/chinese_L-12_H-768_A-12.zip)**并解压到/Users/luyao/Desktop/bert_learn/chinese_L-12_H-768_A-12
- run_classifier.py是Bert自带的程序，在这里我把自己写的放在了run_classifier_0214.py中
- 添加SelfProcessor类，修改了get_train_examples等三个方法，功能参考Bert自带的那些processor即可
- 在main()中增加自己定义的这个processor到字典中


## 训练
- 定义环境变量
>export BERT_BASE_DIR=/Users/luyao/Desktop/bert_learn/chinese_L-12_H-768_A-12
>export MAYI_DIR=/Users/luyao/Desktop/bert_learn
>查看环境变量是否生效 echo $BERT_BASE_DIR 等
- 进行训练(当然可以修改如num_train_epochs等参数)
> python run_classifier_0214.py \\  
  ---task_name=MAYI \\  
  ---do_train=true \\  
  ---do_eval=true \\  
  ---do_predict=false \\  
  ---data_dir=\$MAYI_DIR/MAYI  \\  
  ---vocab_file=\$BERT_BASE_DIR/vocab.txt \\  
  ---bert_config_file=\$BERT_BASE_DIR/bert_config.json \\  
  ---init_checkpoint=\$BERT_BASE_DIR/bert_model.ckpt \\  
  ---max_seq_length=128 \\  
  ---train_batch_size=32 \\  
  ---learning_rate=2e-5 \\  
  ---num_train_epochs=1.0 \\  
  ---output_dir=./tmp/mayi_output/       
   output_dir这个目录可以自动生成，不必手动添加

- 结果在output_dir的eval_results.txt中
>INFO:tensorflow:***** Eval results *****
INFO:tensorflow:  eval_accuracy = 0.75
INFO:tensorflow:  eval_loss = 0.59076154
INFO:tensorflow:  global_step = 4
INFO:tensorflow:  loss = 0.59076154

## 预测
- 定义环境变量
>export TRAINED_CLASSIFIER=/Users/luyao/Desktop/bert_learn/fine/tuned/classifier
使用官方上的demo时，这里有错。这里应该选择train之后生成的目录。
tensorflow.python.framework.errors_impl.NotFoundError: Unsuccessful TensorSliceReader constructor: Failed to find any matching files for /Users/luyao/Desktop/bert_learn/fine/tuned/classifier

>改成    
export TRAINED_CLASSIFIER=./tmp/mayi_output  这是上一步训练后的输出目录

- 进行预测
>python run_classifier_0214.py \\  
  ---task_name=MAYI \\  
  ---do_predict=true \\  
  ---data_dir=\$MAYI_DIR/MAYI \\  
  ---vocab_file=\$BERT_BASE_DIR/vocab.txt \\  
  ---bert_config_file=\$BERT_BASE_DIR/bert_config.json \\  
  ---init_checkpoint=\$TRAINED_CLASSIFIER \\  
  ---max_seq_length=128 \\  
  ---output_dir=./tmp/mayi_output/
  
- 结果在output_dir的test_results.tsv中，第一列为label=0的概率，第二列为label=1的概率
>0.9811994	0.018800637  
0.9783619	0.021638134  
0.97456795	0.025432047  
0.9740146	0.025985414  
0.50997955	0.49002045  
...
