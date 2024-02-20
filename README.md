from flask import Flask, request, jsonify
import json
import os

app = Flask(__name__)

# Set the file path for storing events
events_file = "events.json"

# Helper function to load events from file
def load_events():
    if os.path.exists(events_file):
        with open(events_file, 'r') as f:
            events = json.load(f)
        return events
    else:
        return []

# Helper function to save events to file
def save_events(events):
    with open(events_file, 'w') as f:
        json.dump(events, f, indent=2)

# Endpoint to create a new event
@app.route('/events', methods=['POST'])
def create_event():
    data = request.get_json()

    # Check if required fields are present
    if 'title' not in data or 'description' not in data or 'start_time' not in data or 'end_time' not in data:
        return jsonify({"error": "Missing required fields"}), 400

    # Load existing events
    events = load_events()

    # Add the new event
    events.append(data)

    # Save updated events
    save_events(events)

    return jsonify({"message": "Event created successfully"}), 201

# Endpoint to get all events
@app.route('/events', methods=['GET'])
def get_events():
    events = load_events()

    # Sort events by start_time
    sorted_events = sorted(events, key=lambda x: x['start_time'])

    return jsonify(sorted_events)

# Endpoint to update an existing event
@app.route('/events/<int:event_id>', methods=['PUT'])
def update_event(event_id):
    data = request.get_json()

    events = load_events()

    # Check if the event_id is valid
    if event_id < 0 or event_id >= len(events):
        return jsonify({"error": "Invalid event ID"}), 404

    # Update the event with new data
    events[event_id].update(data)

    # Save updated events
    save_events(events)

    return jsonify({"message": "Event updated successfully"})

# Endpoint to delete an event
@app.route('/events/<int:event_id>', methods=['DELETE'])
def delete_event(event_id):
    events = load_events()

    # Check if the event_id is valid
    if event_id < 0 or event_id >= len(events):
        return jsonify({"error": "Invalid event ID"}), 404

    # Remove the event
    deleted_event = events.pop(event_id)

    # Save updated events
    save_events(events)

    return jsonify({"message": "Event deleted successfully", "deleted_event": deleted_event})

if __name__ == '__main__':
    app.run(debug=True)
