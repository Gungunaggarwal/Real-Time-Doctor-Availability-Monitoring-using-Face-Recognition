import cv2
import face_recognition
import os
import numpy as np
from datetime import datetime, timedelta

def load_known_faces(path="doctors"):
    known_encodings = []
    known_names = []

    for file in os.listdir(path):
        if file.endswith(('.jpg', '.png')):
            image_path = os.path.join(path, file)
            image = face_recognition.load_image_file(image_path)
            encoding = face_recognition.face_encodings(image)
            if encoding:
                known_encodings.append(encoding[0])
                known_names.append(os.path.splitext(file)[0])
    
    return known_encodings, known_names

status_log = {}        
last_seen_time = {}    

ABSENT_THRESHOLD = timedelta(seconds=10)

def mark_present(name):
    now = datetime.now()
    last_seen_time[name] = now
    if status_log.get(name) != "Present":
        print(f"[INFO] {name} marked Present at {now.strftime('%H:%M:%S')}")
        status_log[name] = "Present"

def mark_absent(name):
    if status_log.get(name) != "Absent":
        print(f"[INFO] {name} marked Absent at {datetime.now().strftime('%H:%M:%S')}")
        status_log[name] = "Absent"

def main():
    known_encodings, known_names = load_known_faces()


    for name in known_names:
        status_log[name] = "Absent"
        last_seen_time[name] = datetime.min

    print("[INFO] Starting camera...")
    cap = cv2.VideoCapture(0)

    while True:
        ret, frame = cap.read()
        if not ret:
            break

        small_frame = cv2.resize(frame, (0, 0), fx=0.25, fy=0.25)
        rgb_small_frame = cv2.cvtColor(small_frame, cv2.COLOR_BGR2RGB)

        face_locations = face_recognition.face_locations(rgb_small_frame)
        face_encodings = face_recognition.face_encodings(rgb_small_frame, face_locations)

        seen_in_frame = set()

        for face_encoding, face_location in zip(face_encodings, face_locations):
            matches = face_recognition.compare_faces(known_encodings, face_encoding)
            face_distances = face_recognition.face_distance(known_encodings, face_encoding)

            name = "Unknown"

            if True in matches:
                best_match_index = np.argmin(face_distances)
                name = known_names[best_match_index]
                seen_in_frame.add(name)
                mark_present(name)

            top, right, bottom, left = [i * 4 for i in face_location]  # scale back
            cv2.rectangle(frame, (left, top), (right, bottom), (0, 255, 0), 2)
            cv2.putText(frame, name, (left, top - 10), cv2.FONT_HERSHEY_SIMPLEX, 0.9, (255, 255, 255), 2)

        # Check for absentees
        current_time = datetime.now()
        for name in known_names:
            if name not in seen_in_frame:
                if current_time - last_seen_time[name] > ABSENT_THRESHOLD:
                    mark_absent(name)

        # Display status
        y_offset = 30
        for name in known_names:
            status = status_log[name]
            cv2.putText(frame, f"{name}: {status}", (10, y_offset), cv2.FONT_HERSHEY_SIMPLEX, 0.7,
                        (0, 255, 0) if status == "Present" else (0, 0, 255), 2)
            y_offset += 25

        cv2.imshow('Doctor Presence Monitor', frame)

        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

    print("\n[INFO] Final Status Log:")
    for name, status in status_log.items():
        print(f"{name}: {status}")

    cap.release()
    cv2.destroyAllWindows()

if __name__ == "__main__":
    main()
