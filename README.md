from flask import Flask, request, jsonify
import uuid
import time

app = Flask(__name__)

# In-memory database to store events and alerts
events_db = []
alerts_db = []

# Mock rule function
def apply_rule(events):
    # Implement your rule logic here
    # For this example, let's assume the rule triggers if there are more than 3 events in the past 5 minutes
    if len(events) > 3:
        return True
    return False

def generate_alert(events):
    alert_id = str(uuid.uuid4())
    alert = {
        "id": alert_id,
        "timestamp": int(time.time()),
        "events": events
    }
    alerts_db.append(alert)
    return alert

@app.route('/event', methods=['POST'])
def create_event():
    data = request.json
    events_db.append(data)
    return jsonify({"message": "Event added successfully"}), 201

@app.route('/alert/<string:alert_id>', methods=['GET'])
def get_alert(alert_id):
    alert = next((alert for alert in alerts_db if alert["id"] == alert_id), None)
    if alert:
        return jsonify(alert)
    return jsonify({"message": "Alert not found"}), 404

# Background task to check for alerts every 5 minutes
def check_alerts():
    while True:
        current_time = int(time.time())
        events_in_last_5_minutes = [event for event in events_db if current_time - event["timestamp"] <= 300]
        if apply_rule(events_in_last_5_minutes):
            new_alert = generate_alert(events_in_last_5_minutes)
            print("New alert generated:", new_alert)
        time.sleep(300)  # Sleep for 5 minutes

if __name__ == '__main__':
    import threading

    # Start the background task in a separate thread
    alert_thread = threading.Thread(target=check_alerts)
    alert_thread.daemon = True
    alert_thread.start()

    app.run(debug=True)
