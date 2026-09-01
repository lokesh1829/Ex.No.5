# EXP 5: Comparing Prompting Techniques Through Engineering Problem-Solving Scenarios

**DATE:** 01-09-2026  
**REGISTER NUMBER:** 212225240080  
**NAME:** LOKESH K  

---

### Aim:
To compare different prompting techniques and evaluate their effectiveness in solving real-world engineering problems by using a problem selected from a project scenario (**AI-Based Smart Traffic Management System**) using prompt chaining across multiple development stages.

---

### AI Tools Required:
* Large Language Model Interface (ChatGPT / Gemini)
* Python 3.x Environment (OpenCV, NumPy)

---

### Project Details & Selected Scenario:
* **Project Title:** AI-Powered Real-Time Adaptive Traffic Management and Congestion Control System
* **Problem Statement:** Fixed-timer traffic signal controllers lead to unnecessary waiting times, fuel wastage, and severe congestion at urban road intersections. The objective is to design an intelligent, vision-based traffic light controller that dynamically adjusts green signal duration based on real-time vehicle density detected from camera feeds.

---

### Prompting Implementations (Prompt Chain):

#### 1. Base Prompt (Naïve Prompting)
* **Prompt:**
  > "How to make an AI traffic light system using python and computer vision?"
* **AI Output:**
  > You can build an AI traffic light system using Python, OpenCV, and a YOLO object detection model. The steps are:
  > 1. Capture live video from intersection cameras.
  > 2. Detect and count vehicles (cars, buses, bikes) using YOLO.
  > 3. Write logic to calculate density.
  > 4. Allocate green light time proportionally to the lane with the highest count.
* **Evaluation:** High-level and generic. Lacks system requirements, state transition rules, and safety fallbacks.

---

#### 2. Requirement Analysis Prompt (Persona Pattern)
* **Prompt:**
  > "Act as a Senior Intelligent Transportation Systems (ITS) Architect. Provide a structured Requirement Analysis for an AI-powered real-time adaptive traffic controller across a 4-way intersection. Include functional requirements, hardware constraints, and safety failsafes."
* **AI Output:**
  > **Functional Requirements:**
  > * Real-time vehicle detection and classification (Cars, Heavy Vehicles, Two-wheelers).
  > * Dynamic Green Time calculation: Minimum 10 seconds, Maximum 60 seconds.
  > * Priority vehicle preemption (Ambulances, Fire Trucks).
  > 
  > **Hardware & Sensor Specs:**
  > * 4x 1080p IP Cameras (RTSP stream @ 30 FPS).
  > * Edge Computing Unit: NVIDIA Jetson Orin Nano / Xavier.
  > * Relay Interfacing: RS485 / GPIO interface to solid-state traffic signal relays.
  > 
  > **Safety Failsafes:**
  > * Fallback to default cyclic timer mode upon camera/AI inference failure.
  > * Minimum yellow transition clearance of 3 to 5 seconds.
* **Evaluation:** Precise, technically rigorous, and covers edge-case safety parameters.

---

#### 3. Architecture & Algorithm Design Prompt (Chain of Thought - CoT)
* **Prompt:**
  > "Design the step-by-step algorithm to dynamically allocate green signal duration across 4 lanes (L1, L2, L3, L4). Let's think step-by-step considering vehicle counts ($N_i$), vehicle weight factors ($w_i$), minimum green time ($T_{\text{min}} = 10\text{s}$), and maximum green time ($T_{\text{max}} = 60\text{s}$)."
* **AI Output:**
  > 1. **Step 1: Vehicle Density Scoring:** For each lane $i \in \{1,2,3,4\}$, compute the weighted traffic score:
  >    $$S_i = (N_{\text{car}} \times 1.0) + (N_{\text{bus/truck}} \times 2.5) + (N_{\text{motorcycle}} \times 0.5)$$
  > 2. **Step 2: Total Intersection Demand:**
  >    $$S_{\text{total}} = \sum_{i=1}^{4} S_i$$
  > 3. **Step 3: Dynamic Green Time Allocation:** If lane $k$ is selected for green signal:
  >    $$T_{\text{green}}(k) = \text{clip}\left( \frac{S_k}{S_{\text{total}}} \times T_{\text{cycle}},\, T_{\text{min}},\, T_{\text{max}} \right)$$
  > 4. **Step 4: Lane Scheduling:** Serve lanes in round-robin sequence with dynamically computed $T_{\text{green}}$. If an emergency vehicle is detected, interrupt and grant immediate green.
* **Evaluation:** Highly structured, mathematically sound, and directly implementable.

---

#### 4. Implementation Prompt (Structured Code Generation)
* **Prompt:**
  > "Write a modular, production-ready Python class `TrafficController` that takes vehicle counts for 4 lanes, calculates dynamic green signal duration based on weighted density, enforces minimum/maximum limits, and returns a formatted JSON dictionary for signal relays."
* **AI Output:**
```python
import json

class TrafficController:
    def __init__(self, min_green=10, max_green=60, total_cycle=120):
        self.min_green = min_green
        self.max_green = max_green
        self.total_cycle = total_cycle
        self.weights = {'car': 1.0, 'bus': 2.5, 'truck': 2.5, 'bike': 0.5}

    def compute_lane_score(self, counts: dict) -> float:
        return sum(counts.get(vtype, 0) * weight for vtype, weight in self.weights.items())

    def calculate_green_times(self, lane_data: dict) -> str:
        scores = {lane: self.compute_lane_score(counts) for lane, counts in lane_data.items()}
        total_score = sum(scores.values())

        green_durations = {}
        if total_score == 0:
            default_time = self.total_cycle // len(lane_data)
            green_durations = {lane: default_time for lane in lane_data}
        else:
            for lane, score in scores.items():
                allocated = int((score / total_score) * self.total_cycle)
                clamped = max(self.min_green, min(allocated, self.max_green))
                green_durations[lane] = clamped

        response = {
            "status": "SUCCESS",
            "lane_scores": scores,
            "allocated_green_seconds": green_durations
        }
        return json.dumps(response, indent=4)

if __name__ == "__main__":
    controller = TrafficController()
    lanes = {
        "Lane_1": {"car": 12, "bus": 2, "bike": 8},
        "Lane_2": {"car": 3, "bus": 0, "bike": 2},
        "Lane_3": {"car": 25, "bus": 4, "bike": 15},
        "Lane_4": {"car": 6, "bus": 1, "bike": 4}
    }
    print(controller.calculate_green_times(lanes))
