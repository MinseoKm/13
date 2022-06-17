# AVAudioPlayer 

**이 앱은 기본적으로 '재생 모드'와 '녹음 모드'를 스위치로 전환할 수 있는 앱입니다. 재생, 일시 정지, 정지를 할 수 있으며 음악이 재생되는 동안 재생 정도가 프로그레스 뷰(Progress View)와 시간으로 표시됩니다.**      
**또한 볼륨 조절도 가능합니다. 녹음 모드에서는 녹음을 할 수 있고 녹음이 되는 시간도 표시할 수 있습니다. 녹음이 종료되면 녹음 파일을, 일시 정지, 정지할 수 있습니다. 그리고 이 두 가지 모드를 스위치로 전환하여 반복적으로 사용할 수도 있습니다.** 

***
```
        slVolume.maximumValue = MAX_VOLUME 
        slVolume.value = 1.0 
        pvProgressPlay.progress = 0 
        
        audioPlayer.delegate = self
        audioPlayer.prepareToPlay()
        audioPlayer.volume = slVolume.value
```
슬라이더(slVolume)의 최대 볼륨을 상수 MAX_VOLUME인 10.0으로 초기화합니다.    
슬라이더(slVolume)의 볼륨을 1.0으로 초기화합니다.       
프로그레스 뷰(Progress View)의 진행을 0으로 초기화합니다. 
audioPlayer의 델리게이트를 self로 합니다.          
prepareToPlay()를 실행합니다.         
audioPlayer의 볼륨을 방금 앞에서 초기화한 슬라이더(slVolume)의 볼륨 값 1,0으로 초기화합니다.         

```
        let min = Int(time/60)
        let sec = Int(time.truncatingRemainder(dividingBy: 60)) 
        let strTime = String(format: "%02d:%02d", min, sec) 
        return strTime
```
재생 시간의 매개변수인 time 값을 60으로 나눈 '몫'을 정수 값으로 변환하여 상수 min 값에 초기화합니다.         
time 값을 60으로 나눈 '나머지' 값을 정수 값으로 변환하여 상수 sec값에 초기화합니다.           
이 두 값을 활용해 "%02d:%02d" 형태의 문자열(String)로 변환하여 상수 strTime에 초기화합니다.                
이 값을 호출한 함수로 돌려보냅니다.            

```
        lblEndTime.text = convertNSTimeInterval2String(audioPlayer.duration)
        lblCurrentTime.text = convertNSTimeInterval2String(0)
```
오디오 파일의 재생 시간인 audioPlayer.duration값을 convertNSTimeInterval2String함수를 이용해 lblEndTime의 텍스트에 출력합니다.               
lblCurrentTime의 텍스트에는 convertNSTimeInterval2String함수를 이용해 00:00가 출력되도록 0의 값을 입력합니다.             

```
        audioPlayer.play() 
        setPlayButtons(false, pause: true, stop: true) 
```
audioPlayer.play함수를 실행해 오다오를 재생합니다.             
[Play] 버튼은 비활성화, 나머지 두 버튼은 활성화합니다.              
    
```
    @objc func updatePlayTime() {
        lblCurrentTime.text = convertNSTimeInterval12String(audioPlayer.currentTime) 
        pvProgressPlay.progress = Float(audioPlayer.currentTime/audioPlayer.duration) 
    }
```
재생 시간인 audioPlayer.currentTime을 레이블 'lblCurrentTime' 에 나타냅니다.           
프로그레스 뷰인 pvProgress Play의 진행 상황에 audioPlayer.currentTime을 audioPlayer.duration으로 나눈 값으로 표시합니다.          

```
        audioPlayer.currentTime = 0 
        lblCurrentTime.text = convertNSTimeInterval12String(0) 
        progressTimer.invalidate() 
```
오디오를 정지하고 다시 재생하면 처음부터 재생해야 허므로 audioPlayer.currentTime을 0으로 합니다.               
재생 시간도 00:00로 초기화하기 위해 convertNSTimeInterval12String을 활용합니다.            
타이머도 무효화합니다.            

```
    func audioPlayerDidFinishPlaying(_ player: AVAudioPlayer, successfully flag: Bool) {
        progressTimer.invalidate()
        setPlayButtons(true, pause: false, stop: false)
    }
```        
타이머를 무효화합니다.            
[play] 버튼은 활성화하고 나머지 버튼은 비활성화합니다.               

> 녹음을 위한 초기화 하기         

```
var audiRecorder : AVAudioRecorder!
var isRecordMode = false
```
audioRecorder 인스턴스를 추가합니다.              
현재 '녹음 모드'라는 것을 나타낼 isRecordMode를 추가합니다. 기본값은 false로 하여 처음 앱을 실행했을 때 '녹음 모드'가 아닌 '재생 모드'가 나타나게 합니다.             

```
 func selectAudioFile(){
        if !isRecordMode{
             audioFile = Bundle.main.url(forResource: "Sicilian_Breeze", withExtension: "mp3")
        }else {
            let documentDirectory = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
            audioFile = documentDirectory.appendingPathComponent("recordFile.m4a")
        }
    }
```
재생 모드일 때는 오디오 파일인 "Sicilian_Breeze.mp3"가 선택됩니다.         
녹음 모드일 때는 새 파일인 "recordFile.m4a"가 생성됩니다.                

```
 if !isRecordMode {
            initPlay()
            btnRecord.isEnabled = false
            lblRecordTime.isEnabled = false
        }else {
            initRecord()
        }
```
if문의 조건이 '!isRecordMode'입니다. 이는 '녹음 모드가 아니라면'이므로 재생 모드를 말합니다. 따라서 initPlay 함수를 호출합니다.           
조건에 해당하는 것이 재생 모드이므로 [Record]버튼과 재생 시간은 비활성화로 설정합니다.            
조건에 해당하지 않는 경우, 이는 '녹음 모드라면'이므로 initRecord함수를 호출합니다.            

```
        audioRecorder.delegate = self
        slVolume.value = 1.0
        audioPlayer.volume = slVolume.value
        lblEndTime.text = convertNSTimeInterval2String(0)
        lblCurrentTime.text = convertNSTimeInterval2String(0)
        setPlayButtons(false, pause: false, stop: false)
```
AudioRecorder의 델리게이트(Delegate)를 self로 설정합니다.                        
볼륨 슬라이더 값을 1.0으로 설정합니다.                 
audioPlayer의 볼륨도 슬라이더 값과 동일한 1.0으로 설정합니다.                   
총 재생 시간을 0으로 바꿉니다.              
현재 재생 시간을 0으로 바꿉니다.             
[Plat], [Pause] 및 [Stop] 버튼을 비활성화로 설정합니다.               

```
        if sender.isOn {
            audioPlayer.stop()
            audioPlayer.currentTime = 0
            lblRecordTime!.text = convertNSTimeInterval12String(0)
            isRecordMode = true
            btnRecord.isEnabled = true
            lblRecordTime.isEnabled = true
        } else { 
            
           
            [isRecordMode = false 
            isRecordMode = false
            btnRecord.isEnabled = false
            lblRecordTime.isEnabled = false
            lblRecordTime.text = convertNSTimeInterval12String(0)
        }        
        selectAudioFile()
        if !isRecordMode {
            initPlay()
        } else { 
            initRecord()
        }
```
스위치가 [On]이 되었을 때는 '녹음 모드'이므로 오디오를 재생을 중지하고, 현재 재생 시간을 00:00으로 만들고, isRecordMode의 값을 참(true)으로 설정하고, [Record] 버튼과 녹음 시간을 활성화로 설정합니다.             
스위치가 [On]이 아닐 때, 즉 '재생 모드'일 때는 isRecordMode의 값을 거짓(false)으로 설정하고, [Record] 버튼과 녹음 시간을 비활성화하며, 녹음 시간은 0으로 초기화합니다.           
selectAudioFile 함수를 호출하여 오디오 파일을 선택하고, 모드에 따라 초기화할 함수를 호출합니다.           

```
        if (sender as AnyObject).titleLabel?.text == "Record" {
            audioRecorder.record()
            (sender as AnyObject).setTitle("Stop", for: UIControl.State())
        } else {
            audioRecorder.stop()
            (sender as AnyObject).setTitle("Record", for: UIControl.State())
            btnPlay.isEnabled = true
            initPlay()
        }
```
만약에 버튼 이름이 'Record'이면 녹음을 하고 버튼 이름을 'Stop'으로 변경합니다.             
그렇지 않으면 현재 녹음 중이므로 녹음을 중단하고 버튼 이름을 'Stop'으로 변경합니다. 그리고 [Play] 버튼을 활성화하고 방금 녹음한 파일로 재생을 초기화합니다.          

```
        progressTimer = Timer.scheduledTimer(timeInterval: 0.1, target: self, selector: timeRecordSelector, userInfo: nil, repeats: true)
        progressTimer.invalidate()
```
녹할 때 타이머가 작동하도록 progressTimer에 Timer.scheduledTimer함수를 사용하는데, 0.1초 간격으로 타이머를 생성합니다.             
녹음이 중지되면 타이머를 무효화합니다.           
