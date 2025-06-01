# Evaluation-of-the-football-team-s-performance
def evaluate_player(cards, tackles, shots, key_passes, assists, goals):
    if cards not in [0, -1, -2, -3]:
        raise ValueError("Cards must be 0 (no card), -1 (one yellow), -2 (two yellows), or -3 (red)")

    for value, name in zip([tackles, shots, key_passes], ["Tackles", "Shots", "Key passes"]):
        if not -10 <= value <= 10:
            raise ValueError(f"{name} must be between -10 and 10")

    for value, name in zip([assists, goals], ["Assists", "Goals"]):
        if not 0 <= value <= 10:
            raise ValueError(f"{name} must be between 0 and 10")

    score = (
        cards +
        tackles +
        shots +
        key_passes +
        assists * 2 +
        goals * 3
    )

    return score

def classify_score(score):
    if score >= 20:
        return "Excellent"
    elif score >= 10:
        return "Good"
    elif score >= 0:
        return "Poor"
    else:
        return "Very Poor"

# ---------------------
# Get input from the user
try:
    print("Enter the following player stats:")
    cards = int(input("Cards (0 = no card, -1 = one yellow, -2 = two yellows, -3 = red): "))
    tackles = int(input("Tackles (between -10 and 10): "))
    shots = int(input("Shots on target (between -10 and 10): "))
    key_passes = int(input("Key passes (between -10 and 10): "))
    assists = int(input("Assists (0 to 10): "))
    goals = int(input("Goals (0 to 10): "))

    player_score = evaluate_player(cards, tackles, shots, key_passes, assists, goals)
    player_class = classify_score(player_score)

    print(f"\n✅ Player Total Score: {player_score}")
    print(f"📊 Performance: {player_class}")

except ValueError as e:
    print(f"\n❌ Input Error: {e}")
