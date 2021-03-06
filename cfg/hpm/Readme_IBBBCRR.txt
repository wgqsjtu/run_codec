1.增加一个参数lib_pic_update,当该值为1时，自适应的在低延时下选择更新知识图像；当该值为0时，关闭IBBBCRR配置。
2.i_period，IBBBCRR通测条件中设为2s（帧率25则该值设为50）
3.skip_frames_when_extract_libpic，该值表示提取知识图像时跳过的帧数,在IBBBCRR配置下，该值应当小于等于skip_frames。
4.frames_when_extract_libpic,该值表示提取知识图像使用的帧数，即知识图像对应应用的范围；
  IBBBCRR通测条件中设为250，即串行编码总帧数。
5.qp_offset_rlpic，该值表示RL图像的delta QP。
6.pb_ref_lib,当该值为1时，表示除RL帧以外的P/B帧也可以参考知识图像。
7.rl_ref_lib,当该值为1时，表示RL帧作为P帧参考知识图像；当该值大于1时，表示RL帧作为B帧参考知识图像。
8.lib_in_l0,当该值为0时，表示list0中不存在知识图像；当该值为1时，表示知识图像在list0中顺序在最前面；当该值大于1时，表示知识图像在list0中顺序在最后面。
9.lib_in_l1,当该值为0时，表示list1中不存在知识图像；当该值为1时，表示知识图像在list1中顺序在最前面；当该值大于1时，表示知识图像在list1中顺序在最后面。
10.max_list_refnum,该值表示在一个list中最多可参考的图像数，当当前list参考的短期参考图像数目已经大于等于该值时，则不添加知识图像到该list中。
11.并行运行时，仅需修改变量skip_frames

串行运行脚本示例（Windows Batch）
说明：假设frame rate为5，每2秒为一个分段，编6秒（30帧）
set W=192
set H=192
set QP=50
set FRM=30
set SEQ=Test192
set ORG=D:\WS_DISC_YUV\Test192_192x192.yuv
encoder_app.exe --config .\cfg\encode_IPPPCRR.cfg -i %ORG% -v 1 -w %W% -h %H% -q %QP% -f %FRM% -z 5 -p 10 -o str_%SEQ%_%QP%.bin -r enc_%SEQ%_%QP%_%W%x%H%.yuv -d 8 --internal_bit_depth 8 -s 1 --frames_when_extract_libpic %FRM% > enc_%SEQ%_%QP%.txt
decoder_app.exe -i str_%SEQ%_%QP%.bin -o dec_%SEQ%_%QP%_%W%x%H%.yuv --input_libpics libpic.bin -s > dec_%SEQ%_%QP%.txt

相同结果的分段编码脚本
set W=192
set H=192
set QP=50
set FRM=10
set SEQ=Test192
set ORG=D:\WS_DISC_YUV\Test192_192x192.yuv

::第一段：0到9帧
encoder_app.exe --config .\cfg\encode_IPPPCRR.cfg -i %ORG% -v 1 -w %W% -h %H% -q %QP% -f %FRM% -z 5 -p 10 -o str_%SEQ%_%QP%.bin -r enc_%SEQ%_%QP%_%W%x%H%.yuv -d 8 --internal_bit_depth 8 -s 1 --frames_when_extract_libpic 30 --skip_frames 0 > enc_%SEQ%_%QP%.txt
decoder_app.exe -i str_%SEQ%_%QP%.bin -o dec_%SEQ%_%QP%_%W%x%H%.yuv --input_libpics libpic.bin -s > dec_%SEQ%_%QP%.txt

::第二段：10到19帧
encoder_app.exe --config .\cfg\encode_IPPPCRR.cfg -i %ORG% -v 1 -w %W% -h %H% -q %QP% -f %FRM% -z 5 -p 10 -o str_%SEQ%_%QP%.bin -r enc_%SEQ%_%QP%_%W%x%H%.yuv -d 8 --internal_bit_depth 8 -s 1 --frames_when_extract_libpic 30 --skip_frames 10 > enc_%SEQ%_%QP%.txt
decoder_app.exe -i str_%SEQ%_%QP%.bin -o dec_%SEQ%_%QP%_%W%x%H%.yuv --input_libpics libpic.bin -s > dec_%SEQ%_%QP%.txt

::第三段：20到29帧
encoder_app.exe --config .\cfg\encode_IPPPCRR.cfg -i %ORG% -v 1 -w %W% -h %H% -q %QP% -f %FRM% -z 5 -p 10 -o str_%SEQ%_%QP%.bin -r enc_%SEQ%_%QP%_%W%x%H%.yuv -d 8 --internal_bit_depth 8 -s 1 --frames_when_extract_libpic 30 --skip_frames 20 > enc_%SEQ%_%QP%.txt
decoder_app.exe -i str_%SEQ%_%QP%.bin -o dec_%SEQ%_%QP%_%W%x%H%.yuv --input_libpics libpic.bin -s > dec_%SEQ%_%QP%.txt

注：分成N段编码计算总码率时，每段编码LOG的总码率中均包含了知识图像（L图像）的比特数，因此应当扣除“L图像的比特数 x (N-1)”，即只计算一份知识图像的比特数，知识图像的比特数为以下两行LOG中的数字之和。
Total bits of libpic dependency file(bits) : 24
  Total libpic frames : 1 ,	Total libpic bits(bits) : 5896
