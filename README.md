# 13
13
import UIKit
import AVFoundation // 오디오 재생을 위한 헤더 파일

class ViewController: UIViewController, AVAudioPlayerDelegate, AVAudioRecorderDelegate {
    
    var audioPlayer : AVAudioPlayer! // AVAudioPlayer 인스턴스 변수
    var audioFile : URL! // 재생할 오디오의 파일명 변수
    
    let MAX_VOLUME : Float = 10.0 // 최대 볼륨, 실수형 상수
    
    var progressTimer : Timer! // 타이머를 위한 변수
    
    let timePlayerSelector: Selector = #selector(ViewController.updatePlayTime) // 재생 타이머를 위한 상수 추가
    let timeRecordSelector: Selector = #selector(ViewController.updateRecordTime) // 녹음 타이머를 위한 상수 추가
    
    //미션 이미지(하단)
    @IBOutlet var imgView: UIImageView!
    
    //재생
    @IBOutlet var pvProgressPlay: UIProgressView!
    @IBOutlet var lblCurrentTime: UILabel!
    @IBOutlet var lblEndTime: UILabel!
    @IBOutlet var btnPlay: UIButton!
    @IBOutlet var btnPause: UIButton!
    @IBOutlet var btnStop: UIButton!
    @IBOutlet var slVolume: UISlider!
    
    //녹음
    @IBOutlet var btnRecord: UIButton!
    @IBOutlet var lblRecordTime: UILabel!
    
    var audioRecorder : AVAudioRecorder! //audioRecorder 인스턴스 추가
    var isRecordMode = false //현재가 '녹음 모드' 라는 것을 나타낼 isRecordMode (초기값은 false -> 재생 모드)
    
    //이미지 파일 - 미션
    var imgPlay = UIImage(named: "play.png")
    var imgStop = UIImage(named: "stop.png")
    var imgRecord = UIImage(named: "record.png")
    var imgPause = UIImage(named: "pause.png")
    
    override func viewDidLoad() {
        super.viewDidLoad()
       
        selectAudioFile()
        //녹음 모드가 아니라면, 즉 재생 모드라면 -> initPlay
        if !isRecordMode { // MARK: 재생 모드 일때
            initPlay()
            //재생 모드이기때문에 녹음 버튼, 재생 시간 비활성
            btnRecord.isEnabled = false
            lblRecordTime.isEnabled = false
        } else { // MARK: 녹음 모드 일때
            //녹음 모드 초기화
            initRecord()
        }
        imgView.image = imgStop
    }
    
    //녹음 파일 생성
    //재생 파일에 안 겹치게 모드에 따라 파일 선택하기 위해 함수 생성
    // MARK: 재생 모드와 녹음 모드에 따라 다른 파일을 선택함
    func selectAudioFile() {
        
        //재생모드 일때 사용
        if !isRecordMode {
            
            //재생 모드일 때 재생될 file
            audioFile = Bundle.main.url(forResource: "Sicilian_Breeze", withExtension: "mp3")
            
        } else {
            
            //녹음 모드일때 새파일 생성(recordFile.m4a)
            let documentDirectory = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
            audioFile = documentDirectory.appendingPathComponent("recordFile.m4a")
        }
    }
    
    //녹음을 위한 초기화
    //(녹음btn click)재생 모드에 관한 모든 것들이 초기화
    //포맷 : Apple Lossless
    //음질 : "최대"
    //비트율 : 320000bps(320kbps)
    //오디오 채널 : 2
    //샘플률 : 44100Hz
    // MARK: 녹음 모드의 초기화
    func initRecord() {
        let recordSettings = [
            AVFormatIDKey : NSNumber(value: kAudioFormatAppleLossless as UInt32),
            AVEncoderAudioQualityKey : AVAudioQuality.max.rawValue,
            AVEncoderBitRateKey : 320000,
            AVNumberOfChannelsKey : 2,
            AVSampleRateKey : 44100.0] as [String : Any]
        
        //selectAudioFile 함수에서 정한 audioFile을 URL로 하는 audioRecord인스턴스 생성
        do {
            audioRecorder = try AVAudioRecorder(url: audioFile, settings: recordSettings)
        } catch let error as NSError {
            print("Error-initRecord : \(error)")
        }
        
        //AVAudioRecordDelegate delegate 상속받아준다(ViewController)
        //delegate를 self로 설정
        audioRecorder.delegate = self
        
        slVolume.value = 1.0 //볼륨값 조절
        audioPlayer.volume = slVolume.value //볼륨값 조절
        lblEndTime.text = convertNSTimeInterval12String(0) //총 재생시간 0
        lblCurrentTime.text = convertNSTimeInterval12String(0) //현재 재생시간 0
        setPlayButtons(false, pause: false, stop: false) //Play,Pause,Stop 버튼 비 활성화
        
        //AVAudioSession의 session 생성 후 , 액티브 설정
        let session = AVAudioSession.sharedInstance()
        do {
            try AVAudioSession.sharedInstance().setCategory(.playback, mode: .default)
            try AVAudioSession.sharedInstance().setActive(true)
        } catch let error as NSError {
            print(" Error-setCategory : \(error)")
            
        }
        do {
            try session.setActive(true)
        } catch let error as NSError {
            print(" Error-setActive : \(error)")
        }
    }
    
    // 오디오 재생을 위한 초기화, 재생 모드/녹음 모드 구분 할때 편하기 위해
    // MARK: 재생 모드의 초기화
    func initPlay() {
        // audioPlayer 인스턴스 생성, 파라미터인 오디오 파일 없을때 대비 하여 do-try-catch 문 사용
        do {
            audioPlayer = try AVAudioPlayer(contentsOf: audioFile)
        } catch let error as NSError {
            print("Error-initPlay : \(error)")
        }
        slVolume.maximumValue = MAX_VOLUME // 최대볼륨 10.0 초기화
        slVolume.value = 1.0 // 볼륨 1.0 초기화
        pvProgressPlay.progress = 0 // 프로그레스 뷰 0 초기화
        
        audioPlayer.delegate = self
        audioPlayer.prepareToPlay()
        audioPlayer.volume = slVolume.value
        lblEndTime.text = convertNSTimeInterval12String(audioPlayer.duration) // endTime(총 재생 시간) 초기화
        lblCurrentTime.text = convertNSTimeInterval12String(0) // 00:00
        
        //버튼 제어 코드 간략화
        setPlayButtons(true, pause: false, stop: false)
        
        //        재생, 일시정지, 정지 버튼 제어
     
    }
    
    // 버튼의 동작여부
    // MARK: [재생], [일시 정지], [정지] 버튼을 활성화 또는 비활성화 하는 함수
    func setPlayButtons(_ play: Bool, pause:Bool, stop: Bool) {
        btnPlay.isEnabled = play
        btnPause.isEnabled = pause
        btnStop.isEnabled = stop
    }
    
    //00:00으로 시간 변환해주는 함수
    // MARK: 00:00 형태의 문자열로 변환함
    func convertNSTimeInterval12String(_ time:TimeInterval) -> String {
        let min = Int(time/60) // 60으로 나눈 몫을 정수로 변환하여 return
        let sec = Int(time.truncatingRemainder(dividingBy: 60)) // 60으로 나눈 나머지값을 정수값으로 return
        let strTime = String(format: "%02d:%02d", min, sec) // 위의 두 값을 활용해 "%02d:%02d" 형태의 문자열(String)으로 변환하여 return
        return strTime
    }
    
    // MARK: [재생] 버튼을 클릭하였을 때
    @IBAction func btnPlayAudio(_ sender: UIButton) {
        audioPlayer.play() // 오디오를 재생
        setPlayButtons(false, pause: true, stop: true) // Play버튼은 비활성화, 나머지 두버튼은 활성화 한다
        // 0.1초 간격으로 타이머 생성
        progressTimer = Timer.scheduledTimer(timeInterval: 0.1, target: self, selector: timePlayerSelector, userInfo: nil, repeats: true)
        imgView.image = imgPlay
    }
    
    // MARK: 0.1초마다 호출되며 재생 시간을 표시함
    @objc func updatePlayTime() {
        lblCurrentTime.text = convertNSTimeInterval12String(audioPlayer.currentTime) // 재생시간을 레이블 lblCurrentTime에 표시
        pvProgressPlay.progress = Float(audioPlayer.currentTime/audioPlayer.duration) // 프로그레스 뷰인 pvProgressPlay의 진행상황에 currentTime을 duration으로 나눈 값 표시
    }
    
    // MARK: [일시 정지] 버튼을 클릭하였을 때
    @IBAction func btnPauseAudio(_ sender: UIButton) {
        audioPlayer.pause()
        setPlayButtons(true, pause: false, stop: true)
        imgView.image = imgPause
    }
    
    // MARK: [정지] 버튼을 클릭하였을 때
    @IBAction func btnStopAudio(_ sender: UIButton) {
        audioPlayer.stop()
        audioPlayer.currentTime = 0 //오디오를 정지하고 다시 재생하면 처음부터 재생해야 하므로 0
        lblCurrentTime.text = convertNSTimeInterval12String(0) //재생시간도 00:00로 초기화 하기 위해
        setPlayButtons(true, pause: false, stop: false)
        progressTimer.invalidate() //타이머 무효화
        imgView.image = imgStop
    }
    
    //볼륨 조절
    // MARK:  볼륨 슬라이더 값을 audioplayer.volume에 대입함
    @IBAction func slChangeVolume(_ sender: UISlider) {
        audioPlayer.volume = slVolume.value
    }
    
    //오디오 재생 끝났을 경우 맨 처음 상태로 돌아가는 함수
    // MARK: 재생이 종료되었을 때 호출 됨
    func audioPlayerDidFinishPlaying(_ player: AVAudioPlayer, successfully flag: Bool) {
        progressTimer.invalidate() //타이머 뮤효화
        setPlayButtons(true, pause: false, stop: false) //Play버튼 제외, 나머지 버튼 비활성화
        imgView.image = imgStop
    }
    
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

