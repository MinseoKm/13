# AVAudioPlayer 

**이 앱은 기본적으로 '재생 모드'와 '녹음 모드'를 스위치로 전환할 수 있는 앱입니다. 재생, 일시 정지, 정지를 할 수 있으며 음악이 재생되는 동안 재생 정도가 프로그레스 뷰(Progress View)와 시간으로 표시됩니다. 또한 볼륨 조절도 가능합니다. 녹음 모드에서는 녹음을 할 수 있고 녹음이 되는 시간도 표시할 수 있습니다. 녹음이 종료되면 녹음 파일을, 일시 정지, 정지할 수 있습니다. 그리고 이 두 가지 모드를 스위치로 전환하여 반복적으로 사용할 수도 있습니다.**

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
이 값을 호출한 람수로 돌려보냅니다.

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

> 녹음 구현

```
var audiRecorder : AVAudioRecorder!
var isRecordMode = false
```

// MARK: 스위치를 On/Off하여 녹음 모드인지 재생 모드인지를 결정함
    @IBAction func swRecordMode(_ sender: UISwitch) {
            /*스위치가 On이 되었을 때는 -> 녹음모드
            [재생 중지 / 현재 재생 시간 00:00 / isRecordMode = true
            Record 버튼과 녹음 시간 활성화]*/
        if sender.isOn { // MARK: 녹음 모드 일때
            audioPlayer.stop()
            audioPlayer.currentTime = 0
            lblRecordTime!.text = convertNSTimeInterval12String(0)
            isRecordMode = true
            btnRecord.isEnabled = true
            lblRecordTime.isEnabled = true
            
        } else { // MARK: 재생 모드 일때
            
            /*스위치가 Off이 되었을 때는 -> 재생 모드
            [isRecordMode = false / Record 버튼과 녹음 시간 비활성화 / 녹음시간 0으로 초기화]*/
            isRecordMode = false
            btnRecord.isEnabled = false
            lblRecordTime.isEnabled = false
            lblRecordTime.text = convertNSTimeInterval12String(0)
        }
        
        //selectAudioFile 함수 호출하여 오디오 파일 선택 / 모드에따라 초기화할 함수 호출
        //MARK: 모드에 따라 오디오 파일을 선택 함
        selectAudioFile()
        //MARK: 모드에 따라 재생 초기화 또는 녹음 초기화를 수행
        if !isRecordMode { //MARK: 녹음 모드가 아닐때, 즉 재생 모드일 때
            initPlay()
        } else { //MARK: 녹음 모드 일때
            initRecord()
        }
    }
    
    @IBAction func btnRecord(_ sender: UIButton) {
        // 만약 버튼 이름이 Record 면 녹음을 하고 버튼 이름을 Stop 으로 변경
        if (sender as AnyObject).titleLabel?.text == "Record" {
            //MARK: 버튼이 "Record" 일 때 녹음을 중지함
            audioRecorder.record()
            (sender as AnyObject).setTitle("Stop", for: UIControl.State())
            // 녹음할 때 타이머가 작동하도록 progressTimer에 Time.scheduledTimer 함수를 사용하는데 0.1초 간격으로 타이머 생성
            progressTimer = Timer.scheduledTimer(timeInterval: 0.1, target: self, selector: timeRecordSelector , userInfo: nil, repeats: true)
            imgView.image = imgRecord
        } else { //MARK: 버튼이 "Stop"일 때 녹음을 위한 초기화를 수행함
            // 그렇지 않으면 현재 녹음 중이므로 녹음을 중단하고
            // 버튼이름을 Stop 으로 변경.
            // [Play] 버튼을 활성화 하고 방금 녹음한 파일로 재생을 초기화
            audioRecorder.stop()
            // 녹음이 중지되며 타이머 무효화
            progressTimer.invalidate()
            (sender as AnyObject).setTitle("Record", for: UIControl.State())
            btnPlay.isEnabled = true
            initPlay()
            imgView.image = imgStop
        }
    }
    
    // 타이머에 의해 0.1초 간격으로 이함수를 실행한다. -> 녹음 시간 표시
    //MARK: 0.1초마다 호출되며 녹음 시간을 표시함
    @objc func updateRecordTime() {
        lblRecordTime.text = convertNSTimeInterval12String(audioRecorder.currentTime)
    }
    
}

