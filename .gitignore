import streamlit as st
import requests
import pandas as pd
from streamlit_autorefresh import st_autorefresh

# ----------------------------
# Fetch Quinella (連贏) odds
# ----------------------------
def fetch_quinella_odds(race_no):
    api_url = f"https://racing.stheadline.com/api/raceOdds/latest?raceNo={race_no}&type=quin&rev=2"
    headers = {"User-Agent": "Mozilla/5.0"}
    try:
        r = requests.get(api_url, headers=headers, timeout=5)
        r.raise_for_status()
        json_data = r.json()
    except requests.RequestException as e:
        st.error(f"Error fetching odds: {e}")
        return {}

    quin_odds_list = json_data.get("data", {}).get("quin", {}).get("raceOddsList", [])
    odds = {}
    for item in quin_odds_list:
        h1 = str(item.get("horseNo1"))
        h2 = str(item.get("horseNo2"))
        value = item.get("value")
        if h1 and h2 and value and value > 0:
            odds[f"{h1}-{h2}"] = value
    return odds


# ----------------------------
# Dutching calculator for Banker Quinella
# ----------------------------
def banker_quinella_dutching(banker, selected_others, odds, total_stake):
    valid_pairs = {}
    for other in selected_others:
        pair1 = f"{banker}-{other}"
        pair2 = f"{other}-{banker}"
        if pair1 in odds:
            valid_pairs[pair1] = odds[pair1]
        elif pair2 in odds:
            valid_pairs[pair2] = odds[pair2]

    if not valid_pairs:
        return {}, {}, {}

    inverse_sum = sum(1 / v for v in valid_pairs.values())
    stakes = {p: total_stake / (v * inverse_sum) for p, v in valid_pairs.items()}
    returns = {p: stakes[p] * odds[p] for p in valid_pairs}
    profit = {p: returns[p] - total_stake for p in valid_pairs}

    return valid_pairs, stakes, profit


# ----------------------------
# Streamlit UI
# ----------------------------
st.title("🏇 HKJC Quinella (連贏) Banker Dutching Calculator")

# Sidebar inputs
race_no = st.sidebar.number_input("Select Race Number:", min_value=1, max_value=12, value=1, step=1)
total_stake = st.sidebar.number_input("Total Stake (HKD):", min_value=0.0, value=100.0, step=10.0)
auto_refresh = st.sidebar.checkbox("Auto-refresh every 15s", value=True)

# Auto-refresh
if auto_refresh:
    st_autorefresh(interval=15 * 1000, key="refresh")

# Fetch odds
odds = fetch_quinella_odds(race_no)

if not odds:
    st.warning("No Quinella (連贏) odds available for this race.")
else:
    st.subheader(f"Race {race_no} - Quinella (連贏) Odds")
    df = pd.DataFrame([{"Pair": k, "Odd": v} for k, v in odds.items()])
    st.dataframe(df)

    # List all horses that appear in odds
    all_horses = sorted({int(h) for k in odds.keys() for h in k.split("-")})

    # Banker selection
    banker = st.selectbox("Select your Banker Horse (軸馬):", options=all_horses)

    # Secondary horse selection
    possible_others = [h for h in all_horses if h != banker]
    selected_others = st.multiselect(
        "Select horses to pair with your Banker (副馬):",
        options=possible_others,
        key=f"quinella_others_{race_no}"
    )

    if selected_others:
        valid_pairs, stakes, profit = banker_quinella_dutching(banker, selected_others, odds, total_stake)

        if valid_pairs:
            result_df = pd.DataFrame({
                "Quinella Pair (連贏)": list(valid_pairs.keys()),
                "Odds": [round(valid_pairs[p], 2) for p in valid_pairs],
                "Bet Amount (HKD)": [round(stakes[p], 2) for p in stakes],
                "Expected Return (HKD)": [round(stakes[p] * valid_pairs[p], 2) for p in stakes],
                "Expected Profit (HKD)": [round(profit[p], 2) for p in profit]
            })

            st.subheader("💰 Banker Quinella Dutching Allocation")
            st.dataframe(result_df)

            # Display summary stats
            avg_profit = round(list(profit.values())[0], 2)
            st.success(f"Expected Profit (approx., same for all Quinellas): **{avg_profit} HKD**")

            roi = round((avg_profit / total_stake) * 100, 2)
            st.info(f"ROI: **{roi}%**")

        else:
            st.warning("No valid Quinella odds found for selected combinations.")
    else:
        st.info("Select at least one other horse to create Banker Quinella combinations.")
