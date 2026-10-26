# CSI-WIFI-SENSING
Project OMINOUS
Device-free indoor localization using WiFi Channel State Information (CSI), ESP32-S3 boards, and machine learning.
OMINOUS figures out which part of a room a person is standing in using only the way their body disturbs WiFi signals.

What it does:
Three ESP32 boards mounted onto the walls in a 13x13ft room it captures CSI from ordinary router traffic and stream it to a Python Server. A kNN classifier trained on labeled on each zone in my room which helps predicts where the person is standing in real time and a live web map that shows a dot following them.


How it works:
The firmware part has each board join the WIFI network it enables CSI captures and forwards the raw CSI bytes over UDP. The data collection, stand in each zone for a few minutes while the collector saves labeled samples. The live inference the server keeps on the newest CSI per board classifies at 5 HZ and smooths prediction with a sticky majority vote.

Engineering decisions and what I learned:
1. Geometry did not work but pattern recognition did. My first version of this i tried to estimate positions from signal strength with a weighted centroid like indoor GPS. It failed pretty badly and switching to classifying what area looks like in a stand point from the firmware is what made the tracking work.
2. Fingerprints go stale. After two weeks idle, accuracy collapsed because the radio environment had drifted. Recollecting and retraining fixed it



Limitations: 
It only tracks which zone you’re in rather than exact coordinates, it has a dead zone right next to one receiver, it has to be recalibrated whenever the room setup changes or after long idle periods, and it needs more than three boards to fix those gaps.

Future work: 
The first step would be adding a 4th ESP32 board to cover the dead zone and make tracking smoother. Beyond that, better hardware is what would improve it most, and that’s where I hit my limit: ESP32 CSI is noisy, and finer tracking would need something like mmWave radar or UWB.

Tech stack:
ESP32-S3, C (ESP-IDF), Python, NumPy, scikit-learn, joblib


Running it:
Flash the firmware to each board. Set your WiFi credentials and your PC's IP in main/.
Collect data: python ominous_collect.py and record each zone.
Train: python ominous_train.py
Run live: python ominous_live.py, then open http://localhost:8080

Configuration:
Network credentials and IP addresses are intentionally not committed. Copy the example config, then set your own SSID, password, PC IP, and board IPs before flashing or running.


Repo layout:
firmware/            ESP-IDF project (CSI capture + UDP sender)
ominous_collect.py   labeled data collection
ominous_train.py     model training + evaluation report
ominous_live.py      real-time classifier + web map
