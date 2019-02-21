## Bert  中文文本分类


## 运行环境
- python3.6
- tensorflow1.12

## 数据说明
- 运行时数据所在目录为/Users/luyao/Desktop/bert_learn/MAYI，在这里为了方便起见我把数据一起上传了
- atec_nlp_sim_train_0.6.csv和atec_nlp_sim_test_0.4.csv格式相同，各列分别为"index query1 query2 label"，"\t"分隔。来自网络上蚂蚁金服的公开数据集，是判断文本相似性的数据
- 程序运行中只选择了部分数据(train3000,val500,test100)

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
- 进行训练
> python run_classifier_0214.py \\  
  --task_name=MAYI \\  
  --do_train=true \\  
  --do_eval=true \\  
  --data_dir=\$MAYI_DIR/MAYI  \\  
  --vocab_file=\$BERT_BASE_DIR/vocab.txt \\  
  --bert_config_file=\$BERT_BASE_DIR/bert_config.json \\  
  --init_checkpoint=\$BERT_BASE_DIR/bert_model.ckpt \\  
  --max_seq_length=128 \\  
  --train_batch_size=32 \\  
  --learning_rate=2e-5 \\  
  --num_train_epochs=3.0 \\  
  --output_dir=./tmp/mayi_output/  
  output_dir这个目录可以自动生成，不必手动添加

- 结果在output_dir的eval_results.txt中
>eval_accuracy = 0.8016032  
eval_loss = 0.5034697  
global_step = 281  
loss = 0.5032294  

## 预测
- 定义环境变量
>export TRAINED_CLASSIFIER=/Users/luyao/Desktop/bert_learn/fine/tuned/classifier
使用官方上的demo时，这里有错。应该选择train之后生成的目录。  
tensorflow.python.framework.errors_impl.NotFoundError: Unsuccessful TensorSliceReader constructor: Failed to find any matching files for /Users/luyao/Desktop/bert_learn/fine/tuned/classifier

>改成    
export TRAINED_CLASSIFIER=./tmp/mayi_output  这是上一步训练后的输出目录

- 进行预测
>python run_classifier_0214.py \\  
  --task_name=MAYI \\  
  --do_train=false \\  
  --do_eval=false \\  
  --do_predict=true \\  
  --data_dir=\$MAYI_DIR/MAYI \\  
  --vocab_file=\$BERT_BASE_DIR/vocab.txt \\  
  --bert_config_file=\$BERT_BASE_DIR/bert_config.json \\  
  --init_checkpoint=\$TRAINED_CLASSIFIER \\  
  --max_seq_length=128 \\  
  --output_dir=./tmp/mayi_output/
  
- 结果在output_dir的test_results.tsv中，第一列为label=0的概率，第二列为label=1的概率
>0.54155594	0.45844415  
0.99733293	0.0026670366  
0.98726386	0.012736204   
...  
