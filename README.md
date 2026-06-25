import streamlit as st
import time

# ---------------------------------------------------------
# 1. アプリケーションの初期設定 & データ構造の定義 (クラス図・状態遷移図の再現)
# ---------------------------------------------------------
st.set_page_config(
    page_title="StyleMe - 似合う服・メイク診断", 
    page_icon="👗",
    layout="centered"
)

st.title("👗 StyleMe（スタイルミー）")
st.caption("ソフトウェア工学 PBLプロトタイプ検証用システム")
st.markdown("---")

# データベースの代わりとなるセッション状態（インメモリ管理）の初期化
if "favorites" not in st.session_state:
    st.session_state.favorites = []

# 診断結果が生成されたかどうかを管理する状態フラグ
if "diagnosis_result" not in st.session_state:
    st.session_state.diagnosis_result = None

# ---------------------------------------------------------
# 2. 画面UI（タブ構成）の実装
# ---------------------------------------------------------
tab1, tab2 = st.tabs(["🔍 診断を受ける", "❤️ お気に入り一覧"])

# =========================================================
# タブ1: 診断を受ける画面
# =========================================================
with tab1:
    st.header("似合う服・メイクの診断")
    st.write("あなたの写真と好みの系統から、最適なスタイルをAIが解析します。")
    
    # 【入力】ユーザー入力エリア
    selected_style = st.selectbox(
        "普段の好みの服装の系統を選択してください：", 
        ["フェミニン", "カジュアル", "きれいめ", "モード"]
    )
    
    uploaded_file = st.file_uploader(
        "診断用の顔写真または全身写真をアップロードしてください（JPG / PNG）", 
        type=["jpg", "jpeg", "png"]
    )
    
    # 【処理順序】診断実行イベント
    if st.button("診断を実行する", type="primary"):
        # 【エラーハンドリング】写真未選択時の警告
        if uploaded_file is None:
            st.warning("⚠️ 診断を始めるには、写真をアップロードしてください。")
        else:
            # シーケンス図通りの処理フロー（2秒間のローディングアニメーション）
            with st.spinner("AIによる画像認識・診断ロジックを実行中..."):
                time.sleep(2)  # 2秒間のディレイ
            
            # 【出力】診断結果データの生成（モック処理）
            st.session_state.diagnosis_result = {
                "personal_color": "ブルベ夏",
                "body_shape": "ウェーブ",
                "face_type": "フレッシュ",
                "items": [
                    {
                        "id": 101,
                        "category": "トップス（服）",
                        "brand": "StyleMarket",
                        "name": f"ラベンダーカラーのシアープリーツブラウス ({selected_style}アレンジ)"
                    },
                    {
                        "id": 102,
                        "category": "リップ（メイク）",
                        "brand": "CosmeLab",
                        "name": "ローズピンク系 リキッドルージュ"
                    }
                ]
            }
            st.success("🎉 診断が完了しました！結果を一時データベースに保存しました。")

    # 診断結果が保存されている場合のみ表示（状態の維持）
    if st.session_state.diagnosis_result is not None:
        result = st.session_state.diagnosis_result
        
        st.markdown("### 📊 あなたの診断結果")
        col1, col2, col3 = st.columns(3)
        with col1:
            st.metric(label="パーソナルカラー", value=result["personal_color"])
        with col2:
            st.metric(label="骨格タイプ", value=result["body_shape"])
        with col3:
            st.metric(label="顔タイプ", value=result["face_type"])
            
        st.markdown("### 💡 あなたにおすすめのアイテム")
        st.write("これらのアイテムはクラス図の「提案アイテム」構造に基づいて生成されています。")
        
        # 提案された2つのアイテムをループ表示
        for item in result["items"]:
            with st.container():
                st.write(f"**【{item['category']}】 {item['name']}**")
                st.caption(f"ブランド: {item['brand']} | アイテムID: {item['id']}")
                
                # お気に入り保存処理
                # すでに保存されているかどうかでボタンの挙動を制御
                is_saved = any(fav['id'] == item['id'] for fav in st.session_state.favorites)
                
                if is_saved:
                    st.button("❤️ 保存済み", key=f"saved_{item['id']}", disabled=True)
                else:
                    if st.button("📥 お気に入り保存", key=f"save_{item['id']}"):
                        st.session_state.favorites.append(item)
                        st.toast(f"「{item['name']}」を保存しました！")
                        st.rerun()
            st.write("---")

# =========================================================
# タブ2: お気に入り一覧画面
# =========================================================
with tab2:
    st.header("❤️ 保存したお気に入り一覧")
    st.write("データベース（セッション状態）に登録されているアイテムの一覧です。")
    st.write("---")
    
    # 【出力】データ一覧表示
    if not st.session_state.favorites:
        st.info("お気に入りに登録されたアイテムはまだありません。「診断を受ける」タブから追加してください。")
    else:
        # 保存されているアイテムを一覧表示
        for idx, fav_item in enumerate(st.session_state.favorites):
            col_text, col_btn = st.columns([4, 1])
            
            with col_text:
                st.write(f"**{fav_item['name']}**")
                st.caption(f"カテゴリ: {fav_item['category']} | ブランド: {fav_item['brand']} | ID: {fav_item['id']}")
            
            with col_btn:
                # 【完了条件】削除ボタンの配置とリアルタイム再描画
                if st.button("🗑️ 削除", key=f"del_{fav_item['id']}_{idx}"):
                    st.session_state.favorites.pop(idx)
                    st.toast("お気に入りから削除しました。")
                    st.rerun()
            st.write("---")