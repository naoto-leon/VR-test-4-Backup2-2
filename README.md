# VR-test-4-Backup2-2
VR Oculus-Go Beatsaver 

#### Oculus GoでBeatSaverを作りました。初めてのVR開発でわりかし手間取ったのと時間を掛けたので詳細に記録します。
##### [YouTube](https://youtu.be/9nsiCdtRgEo)
![VR_GIF1](https://user-images.githubusercontent.com/43961147/61991270-e2493480-b088-11e9-8c0b-e08919d54cd5.gif)

***

### 製作動機　

恐らくBeat-Saverをやる為にVRの購入をした人は多いと思う、僕もその一人です。　
しかし、当時の僕はVRに関して全くの無知であり、適正機種がある事など全く知らなかったので、取り敢えず安いお手頃のOculas-Goを買いました。　
##### が、しかし、ご存知の通りOculas-GoではBeat-Saverが出来ない。　　
なんとも悔しい気持ちになったので作ってみました。
結果や満足度に関しては最後にまとめてあります。

***
### 開発環境　

#### Oculas-Go 
□ 3Dof（回転のみ)  
□ 60~72hz 

#### MacBook Pro 
□  バージョン　10.14.5  

#### Unity 
□  バージョン　2019 1.8f1  
□  レンダリング　LWRP  

#### Andoroid SDK 
□ Andoroid 9.0(pie) 

#### JDK 
□ java version "1.8.0_211"  

大事なのはUnityとOculas-Goのスペックと設定。  
まず、3DofというのはVR空間において首の動きのみ連動するという事。（対して屈んだりの動作にも連動している機種を6Dofという）  
正規のBeat-Saverは6Dof対応。  
また、60~72hzはリフレッシュノートであるが、PsVRなどの機種が120hzに対してかなり低い。  
どれくらい低いかというと、スマホと同じくらい。  
なお、スタンドアローン型？(Pcとの連動で動くやつじゃないやつ)。
#### 自分なりにまとめると、スマホのスペックでVRの挙動を実装しなくてはいけない。  
これがかなりネック  
Pc開発時でのFpsが150オーバーにも関わらず実機で確認すると挙動がガタツク場合がある。  

***

### 開発時のエラー　

起きたエラーの中でも印象的だったものについて。  

#### □ Particleが表示されない　
　Unityのバグ、バージョンアップで解決  

#### □ Build成功してもOculas-Goで表示されない（黒い画面）  

  同様のエラーの報告多数有り、原因と解決策は未だ不明のよう。  
  自分は新しくプロジェクトを立て直す事で解決した。    
  ついでに、一度BuildしてアプリとしてOculas-Go内に残したプロジェクトはadbコマンドでなければ消せない（開発者にかなり不評）
　ここら辺のadbコマンドの環境を整えるまでに普段からターミナルを使わない自分はかなり手こずった。    
  (最終的には、プロに教えてもらった)  
  
#### □ Oculas-Go内で録画が出来なくなった件について　

この件に関しては、同様の例がなく根本的な理由、解決方法が未だ不明。  
 スマホとのミラーリングをしての画面録画も試みたが、同様にエラー。  
  
  結局、GoogleのVysorとMediaPlayerを使って録画した。(画質も余り良くないし２画面になった)
  
#### □ Unity Canvasの設定に関して。（表示はされるが。。。）　

　これは完全に自分の考え不足であった。  
 最初は自分の首が動く度にUIが動き不快であり（エースコンバットのような標準を常に首の動きとリンクさせるには良いかもしれないが）  
 なんとか改良したかった。  
 
 Canvasの設定をWorld座標にする事で解決した。（FPSでのUIの挙動を考えれば上記の挙動は自然である）

***

### 音ゲーとしての実装に関して　

　音ゲーと実装としてまず思いつくのが、Jsonやcsv?で楽譜(発生場所や時間)を作り、配列にしてfor文内で、再生曲時間からTimeを引いた値と同じ値に処理される関数をIf文で呼び出すような作り方で、この方法（若干違うけどJsonとか使うのは概ね同じ）を紹介しているサイトは多い。
 
 しかし、正直上記の方法は面倒くさいし、もっと手軽に作りたい。(恐らく分業には適した作り方だと思う)
 
 ということで、BPMを利用して実装を試みた。
 
 大体こんな感じ。
 
 
<img width="330" alt="VR1" src="https://user-images.githubusercontent.com/43961147/61994258-83e57b80-b0b3-11e9-9c68-52176af468cc.png">


実際の実装には上記のシステムをUIで可視化さらに、再生している音楽のBeatを測定するUIのついたgeekdrumsさんのMusicEngineを弄って利用しました。  
##### [geekdrums/MusicEngine](https://github.com/geekdrums/MusicEngine)

***

### VRとしての組み立て　

導入の組み立て(アクティブなコントローラの取得まで)はググれば出てくる。  

基本的にはRaycastを使うがRaycastHitの情報で組み立てるとポイントが当たったら消すという切ってる感が全くなくなる為（第一段階ではTagを使用しての実装を試みた）

      if (hitInfo.collider.tag == "TagName")
          {  Destroy(hitInfo.collider.gameObject);}
        
        
  こんな感じ  
  
  繰り返すが、これだと切ってる感が全くない為下記の方法を試してみた。

<img width="400" alt="VR2" src="https://user-images.githubusercontent.com/43961147/61995139-0de71180-b0bf-11e9-99f0-61f2abe626f0.png">

    if (Vector3.Angle(pointer.position - previousPos, hitInfo.transform.up) > 130f) { 
     Destroy(hitInfo.transform.gameObject);
     }
 
 実際のコードでは130度以上で消すようにした。  
 また、荒い方法ではあるが下からの処理(下からの振り上げへの反応)はマイナスの処理で実装した。(角度及びhitinfoのベクトル)
 
 ***
 
 ### 音とのマッチング　
 
 音とのマッチング性を高める為にゲームそのものにオーディオビジュアルの側面を持たせた。　　
 
####  □ スペクトラムに合わせてParticle、シェーダーを動かす。 

unityでは下記の一文でスペクトラムの取得が可能であり  
    AudioListener.GetSpectrumData(spectrum, 0, FFTWindow.Rectangular);  
 これを下記のようなfor文で処理する事ができる。  
    for (int i = 1; i < spectrum.Length - 1; i++)  

こんな感じ
 ![Spectram](https://user-images.githubusercontent.com/43961147/61995838-61aa2880-b0c8-11e9-9cf0-0f76c6c9f892.gif)

プロジェクトファイルに入れてないので一応記載しとく  
適当なgameobjectにぶっこんでインスペクターからCubeいれれば動く  

    
    using UnityEngine;
    [RequireComponent(typeof(AudioSource))]
    ublic class AudioSourceGetSpectrumDataExample : MonoBehaviour
    {
    public Transform[] cubes;
    public float scale;
    void Update()
    {
        float[] spectrum = new float[256];
        AudioListener.GetSpectrumData(spectrum, 0, FFTWindow.Rectangular);
        for (int i = 0; i < cubes.Length; i++)
        {
            var cube = cubes[i];
            var localScale = cube.localScale;
            localScale.y = spectrum[i] * scale;
            cube.localScale = localScale;
           }
        }
    }   
    

 実装にはkeijiroさんのunity-audio-spectrumを利用させていただいた。　
 ##### [keijiro/unity-audio-spectrum]( https://github.com/keijiro/unity-audio-spectrum)

 ##### Particleの取得はジェネリック型ではない為GetComponent(typeof(ParticleSystem))で取得。      
 ##### シェーダーに関してはGetComponent<MeshRenderer>();でマテリアルにアクセスし、SetFloatでシェーダーで設定した変数を取得。
  
  ***
  
   ### ビジュアライズに関して　
   
   可能限り使用したParticle Shader諸々を紹介します。

***

  ### 開発のヒント
