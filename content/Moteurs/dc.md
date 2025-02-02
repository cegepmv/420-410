+++
title = 'Moteurs DC'
date = 2025-01-27T16:59:50-05:00
draft = true
weight = 31
+++


# Fonctionnement

# Contrôleur 
On utilise le L293D

# Exemple

```python
import pigpio
import time

# Broches GPIO
ENABLE = 17
IN1 = 27
IN2 = 22

# Initialisation
pi = pigpio.pi()
pi.set_mode(ENABLE,pigpio.OUTPUT)
pi.set_mode(IN1,pigpio.OUTPUT)
pi.set_mode(IN2,pigpio.OUTPUT)

try:
    # Faire tourner le moteur à 100%
    pi.write(IN1, 1)
    pi.write(IN2, 0)
    pi.set_PWM_dutycycle(ENABLE, 255)
    time.sleep(2)
    
    # Changer de direction
    pi.write(IN1, 0)
    pi.write(IN2, 1)
    time.sleep(2)    
    
    # Diminuer le régime à 50%
    pi.set_PWM_dutycycle(ENABLE, 128)
    time.sleep(2)
    
    # Arrêter    
    pi.write(IN1,0)
    pi.write(IN2,0)
    pi.write(ENABLE,0)

except KeyboardInterrupt:
    pi.stop()
```