# Choosing Transcription Methods

## What's the difference?

When transcribing speech to text, Azure provides two methods: Real-time and Batch. Both have pros and cons depending on use-case:

**Real-time Transcription**

Real-time transcription, as the name suggests, transcribes audio as it's happening. This method is typically used for live events, meetings, or any scenario where immediate transcription is required. It allows applications to provide live captions for streaming media, transcribe phone conversations, and enable voice assistants to create natural, human-like conversational interfaces.

The primary advantage of real-time transcription is its immediacy; it can provide near-instantaneous transcriptions, making it ideal for live events or interactive applications. However, the downside is that the accuracy of real-time transcription might be slightly lower compared to batch transcription due to the time constraint. Also, handling large volumes of data in real time can be more resource-intensive.

**Batch Transcription**

On the other hand, batch transcription is used to transcribe a large amount of audio in storage. In this method, you point to audio files with a URI and asynchronously receive transcription results. This is particularly useful for applications that need to transcribe audio in bulk, such as transcriptions, captions, or subtitles for pre-recorded audio.

The main advantage of batch transcription is that it can handle a large number of submitted transcriptions concurrently, reducing the overall turnaround time. It also allows for more intensive processing, which can lead to higher accuracy, especially for complex or specific language. However, the downside is that batch transcription is not immediate; you must wait for the transcription to complete before you can access the results.



We'll be starting off with an example project to transcibe call recordings and provide brief summaries, so we need to transcribe multiple files at once where accuracy is important, and it doesn't need to happen as the speech is occurring. This is a perfect use-case for batch transcription, so I'll start by focusing on that method.&#x20;
