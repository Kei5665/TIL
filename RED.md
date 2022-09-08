## 流れ
ある議題について質問される
選択肢は賛成.反対
質問はジャンルを持っている
答えによってジャンルを持っている政党が何かに足される
何問か繰り返す
結果的に点数の高かった政党をランキングされる。

ゲーム開始と同時に
1. Game.new
2. current_game = Game.find(params[:game_id])
3. PoliticalPartie.all each do party
      current_game.points.new(political_party_id: party.id)
    end

