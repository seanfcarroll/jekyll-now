

class DeckSummary

  def initialize(deck_id,user_id)
    @deck_id = deck_id.to_i
    @user_id = user_id.to_i
    @data = data
  end

  def id
    @data[0].id
  end

  def deck
    @data[0].deck
  end

  def card_count
    @data[0].card_count
  end

  def last_deck_test_correct
    @data[0].correct
  end

  def last_deck_test_incorrect
    @data[0].incorrect
  end

  def last_deck_test_count
    last_deck_test_correct + last_deck_test_incorrect
  end

  def last_deck_test_pct
    return 0 if last_deck_test_count == 0
    (last_deck_test_correct / last_deck_test_count) * 100
  end

  def total_card_views
    @data[0].total_card_views
  end

  def total_deck_tests_correct
    @data[0].total_correct
  end

  def total_deck_tests_incorrect
    @data[0].total_incorrect
  end

  def total_deck_tests_count
    total_deck_tests_correct + total_deck_tests_incorrect
  end

  def total_deck_tests_pct
    return 0 if total_deck_tests_count == 0
    (total_deck_tests_correct / total_deck_tests_count) * 100
  end

  private
  def data
    sql = %Q[
      SELECT d.id,
         d.deck,
         f.card_count,
         dt_vw.id,
         coalesce(dt_vw.row_num,1)             AS row_num,
         COALESCE(dt_vw.card_views,0)          AS card_views,
         COALESCE(dt_vw.correct,0)             AS correct,
         COALESCE(dt_vw.incorrect,0)           AS incorrect,
         coalesce(dt_vw.total_card_views,0)    AS total_card_views,
         coalesce(dt_vw.total_correct,0)       AS total_correct,
         COALESCE(dt_vw.total_incorrect,0)     AS total_incorrect
  FROM (SELECT f.deck_id,
               count(f.*) AS card_count
          FROM cards f
         GROUP BY f.deck_id) f,
       decks d
       LEFT OUTER JOIN (SELECT ROW_number() OVER w AS row_num,
                               dt.id,
                               dt.deck_id,
                               dt.id as card_views,
                               dt.id as correct,
                               dt.id as incorrect,
                               SUM(dt.id)  OVER w AS total_card_views,
                               SUM(dt.id)     OVER w AS total_correct,
                               SUM(dt.id)   OVER w AS total_incorrect
                        FROM deck_tests dt
                        WHERE dt.user_id = #{@user_id}
                        WINDOW w AS (PARTITION by dt.deck_id, dt.user_id ORDER BY dt.id desc)) dt_vw
        ON dt_vw.deck_id = d.id
  WHERE d.id = #{@deck_id}
  AND d.id = f.deck_id
  AND row_num = 1
  ORDER BY d.id
    ]
    Deck.find_by_sql(sql)
  end
end
