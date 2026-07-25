# ずんだもんボイス（VOICEVOX音声）の作り方メモ

2026-07-25 会社PC（クラウド環境）のClaudeが確立した手順。
「ずんだもんニュース7月25日 V01」（zundamon/2026-07-25.html）はこの方法で音声を埋め込んだ。
クラウド環境にはVOICEVOXアプリが無くても、VOICEVOX Core（無償公開）で本物のずんだもんの声を合成できる。

## 1. 必要ファイルのダウンロード（すべてGitHubの直リンク。APIは使わない）

バージョンは 0.16.4 時点。新しい版が出たら数字を読み替える
（最新版の確認はWebFetchで https://github.com/VOICEVOX/voicevox_core/releases/latest を見る）。

```bash
# Python バインディング（wheel）※ファイル名はこの形式のまま保存すること
curl -sSfL -O https://github.com/VOICEVOX/voicevox_core/releases/download/0.16.4/voicevox_core-0.16.4-cp310-abi3-manylinux_2_34_x86_64.whl

# 推論ランタイム
curl -sSfL -o ort.tgz https://github.com/VOICEVOX/onnxruntime-builder/releases/download/voicevox_onnxruntime-1.17.3/voicevox_onnxruntime-linux-x64-1.17.3.tgz
tar xzf ort.tgz

# 音声モデル（0.vvm に ずんだもん が入っている）
curl -sSfL -O https://github.com/VOICEVOX/voicevox_vvm/releases/download/0.16.4/0.vvm

# Open JTalk 辞書（SourceForgeはプロキシで落ちるのでGitHubミラーを使う）
curl -sSfL -o dict.tar.gz https://github.com/r9y9/open_jtalk/releases/download/v1.11.1/open_jtalk_dic_utf_8-1.11.tar.gz
tar xzf dict.tar.gz
```

つまずきポイント:
- 公式ダウンローダー（download-linux-x64）はGitHub APIを叩くので、
  この環境では「Bad credentials」や「API rate limit」で失敗する。**直リンクで落とすこと**。
- github.com のHTMLページはプロキシ403。**アセット直リンクは通る**。ページ情報はWebFetchで見る。

## 2. 合成（Python）

```bash
python3 -m venv venv
./venv/bin/pip install voicevox_core-0.16.4-*.whl lameenc   # lameenc = ffmpeg不要のmp3エンコーダ
```

```python
from voicevox_core.blocking import Onnxruntime, OpenJtalk, Synthesizer, VoiceModelFile
import lameenc, wave, io, base64

ort = Onnxruntime.load_once(filename="voicevox_onnxruntime-linux-x64-1.17.3/lib/libvoicevox_onnxruntime.so")
syn = Synthesizer(ort, OpenJtalk("open_jtalk_dic_utf_8-1.11"))
with VoiceModelFile.open("0.vvm") as m:
    syn.load_voice_model(m)

wav = syn.tts("やっほーなのだ。", 3)   # スタイルID 3 = ずんだもん ノーマル
# ID: あまあま=1 / ノーマル=3 / セクシー=5 / ツンツン=7（四国めたん ノーマル=2）

w = wave.open(io.BytesIO(wav))         # 24kHz/16bit/モノラルのWAVが返る
enc = lameenc.Encoder(); enc.set_bit_rate(48); enc.set_in_sample_rate(w.getframerate())
enc.set_channels(1); enc.set_quality(2)
mp3 = bytes(enc.encode(w.readframes(w.getnframes()))) + bytes(enc.flush())
b64 = base64.b64encode(mp3).decode()
```

## 3. HTMLへの埋め込み（つぐのずんだ方式）

- `const VO = {"0":"<base64>", "1":...}` を台本の行番号キーで埋め込む
- 再生は `new Audio('data:audio/mpeg;base64,'+VO[i]).play()`（行送りのたびに前の音声をpause）
- スマホの自動再生制限があるため、**最初のタップ以降に再生**する設計にする
- 48kbpsモノラルで1行あたり40〜100KB程度。25行（約4分半）で合計約1.6MB

## 4. クレジット表記（利用規約）

VOICEVOX音声モデルの利用規約により、**「VOICEVOX:ずんだもん」のクレジット表記が必要**。
タイトル画面かフッターに必ず入れること（つぐのずんだのタイトル画面と同じ流儀）。
規約: https://github.com/VOICEVOX/voicevox_vvm の TERMS.txt

## 5. 関連メモ

- つぐのずんだ（tsugu-no-zunda リポジトリ）のクレジットは
  「VOICEVOX:ずんだもん / 四国めたん / 東北イタコ　効果音:効果音ラボ　音楽:Suno」。
  ビルドスクリプト本体はGitHubには無い（おそらく自宅PC側にある）。
- 東北イタコなど他の声を使う場合は、別の番号のvvmに入っているので
  metasを列挙して探す（VoiceModelFile.open(f).metas）。
