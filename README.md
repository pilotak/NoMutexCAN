# NoMutexCAN
Standard mbed CAN library doesn't allow you to read messages in ISR. And when you try to use EventQueue (`can.attach(eQueue.event(canListen), CAN::RxIrq);`) it will give you `ISR overflow error` so the only chance is to override mutex, that's what this library do.

```cpp
#include "mbed.h"
#include "NoMutexCAN.h"

CircularBuffer<CANMessage, 32> queue;
NoMutexCAN can(PB_8, PB_9);

void canListen() {  // ISR
    CANMessage msg;

    if (can.read(msg)) {
        queue.push(msg);
    }
}

int main(void) {
    can.frequency(250000);
    can.mode(CAN::Normal);

    can.attach(&canListen, CAN::RxIrq);

    while (1) {
        while (!queue.empty()) {
            CANMessage msg;
            queue.pop(msg);
            printf("ID: %X\n", msg.id);
        }
    }
}
```