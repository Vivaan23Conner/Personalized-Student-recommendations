import pandas as pd
import numpy as np
from flask import Flask, request, jsonify
from collections import defaultdict

# Initialize Flask App
app = Flask(__name__)

# Sample function to load data (Replace this with actual API calls for live data)
def load_data():
    # Simulate Current Quiz Data
    current_quiz_data = pd.DataFrame({
        "question_id": [1, 2, 3, 4],
        "topic": ["Math", "Science", "Math", "History"],
        "difficulty": ["Easy", "Medium", "Hard", "Medium"],
        "is_correct": [1, 0, 1, 0]
    })

    # Simulate Historical Quiz Data
    historical_data = pd.DataFrame({
        "quiz_id": [101, 102, 103, 104, 105],
        "topic": ["Math", "Science", "History", "Math", "Science"],
        "score": [80, 60, 70, 90, 50],
        "difficulty_accuracy": [{"Easy": 0.8, "Medium": 0.7}, {"Easy": 0.6, "Medium": 0.5}, {}, {"Hard": 0.9}, {"Medium": 0.5}]
    })

    return current_quiz_data, historical_data

# Function to analyze data and generate recommendations
def analyze_and_recommend(user_id):
    current_quiz, historical_data = load_data()

    # Analyze performance by topic
    topic_performance = current_quiz.groupby("topic")["is_correct"].mean()

    # Analyze difficulty accuracy trends
    difficulty_accuracy = defaultdict(list)
    for entry in historical_data["difficulty_accuracy"]:
        for difficulty, accuracy in entry.items():
            difficulty_accuracy[difficulty].append(accuracy)
    difficulty_summary = {k: np.mean(v) for k, v in difficulty_accuracy.items()}

    # Identify weak areas
    weak_topics = topic_performance[topic_performance < 0.5].index.tolist()

    # Recommendations
    recommendations = {
        "focus_topics": weak_topics,
        "difficulty_to_practice": [k for k, v in difficulty_summary.items() if v < 0.6]
    }

    # Generate persona insights
    strengths = topic_performance[topic_performance >= 0.8].index.tolist()
    persona = "Diligent Learner" if len(strengths) > len(weak_topics) else "Needs Improvement"

    recommendations["persona"] = persona
    recommendations["strengths"] = strengths

    return recommendations

# API Endpoint to get recommendations
@app.route('/recommend', methods=['POST'])
def recommend():
    user_id = request.json.get("user_id")
    if not user_id or not isinstance(user_id, int):
        return jsonify({"error": "Valid User ID is required"}), 400

    recommendations = analyze_and_recommend(user_id)
    return jsonify(recommendations)

if __name__ == "__main__":
    app.run(debug=True)
