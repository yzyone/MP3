### （转载）MP3 转PCM 的 Java方式实现 ###

需求说明：

使用讯飞语音时，默认是传 wav 或 pcm 格式的音频， 传到后台，后台服务 只能识别的音频的格式是mp3，所以需要将 mp3 转成 pcm 。

使用方法：

将传来的 wav 或 pcm 格式的原始音频文件 保存到服务器上，然后转换成 mp3。

( 也可以直接使用流，调用 mp.getAudioInputStream(inputstream ) 方法，而无须保存到服务器上)


```
import java.io.File;
import java.io.IOException;

import javax.sound.sampled.AudioFileFormat;
import javax.sound.sampled.AudioFormat;
import javax.sound.sampled.AudioInputStream;
import javax.sound.sampled.AudioSystem;
import javax.sound.sampled.DataLine;
import javax.sound.sampled.SourceDataLine;

import javazoom.spi.mpeg.sampled.file.MpegAudioFileReader;

/**
 * MP3转PCM Java方式实现
 */
public class AudioUtils {

	private AudioUtils() {
	}


	/**
	 * MP3转换PCM
	 * 
	 * @param mp3filepath 原始文件路径
	 * @param pcmfilepath 转换文件的保存路径
	 * @return
	 * @throws Exception
	 */
	public static boolean convertMP3ToPcm(String mp3filepath, String pcmfilepath) {
		try {
			// 获取文件的音频流，pcm的格式
			AudioInputStream audioInputStream = getPcmAudioInputStream(mp3filepath);
			// 将音频转化为 pcm的格式保存下来
			AudioSystem.write(audioInputStream, AudioFileFormat.Type.WAVE, new File(pcmfilepath));
			return true;
		} catch (IOException e) {
			e.printStackTrace();
			return false;
		}
	}

	/**
	 * 播放MP3
	 * 
	 * @param mp3filepath
	 * @throws Exception
	 */
	public static void playMP3(String mp3filepath) throws Exception {
		// 获取音频为pcm的格式
		AudioInputStream audioInputStream = getPcmAudioInputStream(mp3filepath);

		// 播放
		if (audioInputStream == null) {
			System.out.println("null audiostream");
			return;
		}
		// 获取音频的格式
		AudioFormat targetFormat = audioInputStream.getFormat();
		DataLine.Info dinfo = new DataLine.Info(SourceDataLine.class, targetFormat, AudioSystem.NOT_SPECIFIED);
		// 输出设备
		SourceDataLine line = null;
		try {
			line = (SourceDataLine) AudioSystem.getLine(dinfo);
			line.open(targetFormat);
			line.start();

			int len = -1;
			byte[] buffer = new byte[1024];
			// 读取音频文件
			while ((len = audioInputStream.read(buffer)) > 0) {
				// 输出音频文件
				line.write(buffer, 0, len);
			}

			// Block等待临时数据被输出为空
			line.drain();

			// 关闭读取流
			audioInputStream.close();

			// 停止播放
			line.stop();

		} catch (Exception ex) {
			ex.printStackTrace();
			System.out.println("audio problem " + ex);
		}finally{
			if(line!=null){
				line.close();
			}
		}
	}

	/**
	 * 获取文件的音频流
	 * 
	 * @param mp3filepath
	 * @return
	 */
	private static AudioInputStream getPcmAudioInputStream(String mp3filepath) {
		File mp3 = new File(mp3filepath);
		AudioInputStream audioInputStream = null;
		AudioFormat targetFormat = null;
		AudioInputStream in = null;
		try {

			// 读取音频文件的类
			MpegAudioFileReader mp = new MpegAudioFileReader();
			in = mp.getAudioInputStream(mp3);
			AudioFormat baseFormat = in.getFormat();

			// 设定输出格式为pcm格式的音频文件
			targetFormat = new AudioFormat(AudioFormat.Encoding.PCM_SIGNED, baseFormat.getSampleRate(), 16,
					baseFormat.getChannels(), baseFormat.getChannels() * 2, baseFormat.getSampleRate(), false);

			// 输出到音频
			audioInputStream = AudioSystem.getAudioInputStream(targetFormat, in);
			
		} catch (Exception e) {
			e.printStackTrace();
		}
		return audioInputStream;
	}
}
```

依赖：


	<dependency>
    	<groupId>com.googlecode.soundlibs</groupId>
    	<artifactId>mp3spi</artifactId>
    	<version>1.9.5.4</version>
	</dependency>


原文链接：https://blog.csdn.net/xiaojin21cen/article/details/86591690