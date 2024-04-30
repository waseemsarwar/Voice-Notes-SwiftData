# Voice-Notes-SwiftData
This sample app is about voice to text notes without using any external library/APIs

Please note: App will only work on real device

## Features:
### Voice to text
Recording & speech recognizer
```Swift
    // Configure the audio session for the app.
    let audioSession = AVAudioSession.sharedInstance()
    let inputNode = audioEngine.inputNode

    // try catch to start audio session
    do{
        try audioSession.setCategory(.record, mode: .measurement, options: .duckOthers)
        try audioSession.setActive(true, options: .notifyOthersOnDeactivation)
    }catch{
        print("ERROR: - Audio Session Failed!")
    }

    // Configure the microphone input and request auth
    let recordingFormat = inputNode.outputFormat(forBus: 0)
    inputNode.installTap(onBus: 0, bufferSize: 1024, format: recordingFormat) { (buffer: AVAudioPCMBuffer, when: AVAudioTime) in
        self.recognitionRequest?.append(buffer)
    }

    audioEngine.prepare()

    do{
        try audioEngine.start()
    }catch{
        print("ERROR: - Audio Engine failed to start")
    }

    // Create and configure the speech recognition request.
    recognitionRequest = SFSpeechAudioBufferRecognitionRequest()
    guard let recognitionRequest = recognitionRequest else { fatalError("Unable to create a SFSpeechAudioBufferRecognitionRequest object") }
    recognitionRequest.shouldReportPartialResults = true

    // Create a recognition task for the speech recognition session.
    recognitionTask = speechRecognizer?.recognitionTask(with: recognitionRequest){ result, error in
        if (result != nil){
            self.outputText = (result?.transcriptions[0].formattedString)!
        }
        if let result = result{
            // Update the text view with the results.
            self.outputText = result.transcriptions[0].formattedString
        }
        if error != nil {
            // Stop recognizing speech if there is a problem.
            self.audioEngine.stop()
            inputNode.removeTap(onBus: 0)
            self.recognitionRequest = nil
            self.recognitionTask = nil

        }
    }

```
### Volume keys press observe to start recording note
```Swift
    do {
        try AVAudioSession.sharedInstance().setCategory(AVAudioSession.Category.ambient)
        try session.setActive(true, options: .notifyOthersOnDeactivation)
    } catch {
        print("cannot activate session")
    }
    var initialObservation = true
    progressObserver = session.observe(\.outputVolume) { [self] (session, value) in
        if initialObservation {
            initialObservation = false
            return
        }
        DispatchQueue.main.async {
            self.pressVolume = true
            print("outputVolume")
        }
    }

```
### Persistance storage with SwiftData
Its so simple and easy, just declare your model class with annotation `@Model`
```Swift
import SwiftData

@Model
class Note {
  var transcript: String!
    var date: Date!

  init(transcript: String, date: Date) {
    self.transcript = transcript
    self.date = date
  }
}

```
### iCloud sync
Select the app target, inside `Sign & Capabilities` enable iCloud->CloudKit->Container
<img width="1387" alt="iCloud" src="https://github.com/waseemsarwar/Voice-Notes-SwiftData/assets/1830438/a011af37-a2f1-4187-8052-bc805876edf6">


