By LongGang Pang, from UC Berkeley

BERT应用之《红楼梦》对话人物提取

1. 这个目录包含用于提取对话人物语境的脚本 conversation_extraction.ipynb，
2. 辅助打标签的脚本 label_data_by_click_buttons.ipynb，
3. 提取出的语境文件：honglou.py 
4. 打过标签的训练数据：label_honglou.txt
5. 从打过标签的数据合成百万级别新数据的脚本：augment_data.py
6. 将训练数据转换为BERT/SQUAD可读的脚本：prepare_squad_data.py
7. 以及预测结果文件：
8. res.txt (使用36000组数据训练后的预测结果）；
9. res_1p2million.txt（使用120万组数据训练后的预测结果）。
10. res_crf.txt: 使用条件随机场预测的结果。
11. bert_vs_crf.txt: BERT 与 条件随机场预测结果的不同条目的对比。

对比8和9之后发现使用更多的数据训练所提升的效果有限，比较大的提升是后者在没有答案时，输出是输入的完整拷贝。

摘录 res.txt 中部分内容如下，每一行是一组语境数据以及预测结果，两者用|||分隔：

> 那李嬷嬷听如此说，只得和众人去吃些酒水．这里宝玉又说：  |||  宝玉

> 薛姨妈忙道：  |||  薛姨妈

> 宝钗笑道：  |||  宝钗

> 黛玉磕着瓜子儿，只抿着嘴笑．可巧黛玉的小丫鬟雪雁走来与黛玉送小手炉，黛玉因含笑问他：  |||  黛玉

> 雪雁道：  |||  雪雁

> 黛玉一面接了，抱在怀中，笑道：  |||  黛玉

> 宝玉听这话，知是黛玉借此奚落他，也无回复之词，只嘻嘻的笑两阵罢了．宝钗素知黛玉是如此惯了的，也不去睬他．薛姨妈因道：  |||  薛姨妈因

> 黛玉笑道：  |||  黛玉

> 薛姨妈道：  |||  薛姨妈

> 说话时，宝玉已是三杯过去．李嬷嬷又上来拦阻．宝玉正在心甜意洽之时，和宝黛姊妹说说笑笑的，那肯不吃．宝玉只得屈意央告：  |||  宝玉

> 李嬷嬷道：  |||  李嬷嬷

> 宝玉听了这话，便心中大不自在，慢慢的放下酒，垂了头．黛玉先忙的说：  |||  黛玉

> 一面悄推宝玉，使他赌气，一面悄悄的咕哝说：  |||  宝玉

> 那李嬷嬷不知黛玉的意思，因说道：  |||  李嬷嬷

> 林黛玉冷笑道：  |||  林黛玉

> 李嬷嬷听了，又是急，又是笑，说道：  |||  李嬷嬷

> 宝钗也忍不住笑着，把黛玉腮上一拧，说道：  |||  宝钗

> 薛姨妈一面又说：  |||  薛姨妈

对比 BERT 与 条件随机场（CRF）的预测结果，当语境中没有说话的人，CRF 预测的更好。比如下面这些预测结果（BERT在前,由<开头，CRF在后，由>开头）：
>    40 < 因问：  |||  因

>    42 > 因问：  ||| 


>   112 < 又忙携黛玉之手，问：  |||  黛玉

>   113 < 一面又问婆子们：  |||  一面

>   114 < 说话时，已摆了茶果上来．熙凤亲为捧茶捧果．又见二舅母问他：  |||  二舅母

>   115 ---

>   116 > 又忙携黛玉之手，问：  |||  

>   117 > 一面又问婆子们：  |||  

>   118 > 说话时，已摆了茶果上来．熙凤亲为捧茶捧果．又见二舅母问他：  |||


但是当语境中有多个人物，CRF 会挑出很多个候选，并不像BERT一样，每条语境只挑出一个候选。

>   168 < 黛玉便说了名．宝玉又问表字．黛玉道：  |||  黛玉

>   169 ---

>   170 > 黛玉便说了名．宝玉又问表字．黛玉道：  |||  黛玉宝玉黛玉


而且有很多语境，BERT 能成功识别说话的人，但 CRF 莫名其妙的失败，可能是因为训练语料中没有出现这种模式。
相比来说，BERT 好像对语言理解的更多，比如

>   124 < 当下茶果已撤，贾母命两个老嬷嬷带了黛玉去见两个母舅．时贾赦之妻邢氏忙亦起身，笑回道：  |||  邢氏

>   125 ---

>   126 > 当下茶果已撤，贾母命两个老嬷嬷带了黛玉去见两个母舅．时贾赦之妻邢氏忙亦起身，笑回道：  |||  



1. conversation_extraction.ipynb is used to extract conversation and the context which contains the information of the speaker.

2. honglou.py stores the extracted conversation and context information

3. label_data_by_click_buttons.ipynb is used to extract speakers by converting the context sentence to a large group of buttons.
It is much easier to click on the start and end words of the speaker (which are buttons now) to label the data.
Program will count the position automatically.

4. label_honglou.txt: the labeled speaker and contexted information for about 1562 examples

5. augment_data.py: using data augmentation to construct 1562*1562 examples by combine replacing the original speaker with speakers in different context.

6. dataset.py: use augment_data.py to create training dataset.

7. prepare_squad_data.py: convert the training dataset to json format which can be understood by BERT/SQUAD

8. prediction_results_utf8.py: the prediction results for all the context extracted from hong lou meng.

9. compare_res.py: put the context and prediction results side-by-side for a clearer comparison

10. res.txt and res_1p2million.txt: prediction results using 36000 and 1.2 million training examples correspondingly.
