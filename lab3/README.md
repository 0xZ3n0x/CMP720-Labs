# SafePulse Lab 3 - Reflection Notes

---

## Section 6: Deliberate Timing Failure Experiment

### Experiment A: Printf in the ISR

Adding a long `printf` (~17ms) inside the ISR causes:

- ISR blocking time increases from ~1ms to ~17ms
- Sampling interval becomes irregular (not 200ms anymore)
- Main loop starves - cannot run while ISR executes
- CPU stays busy in ISR, system becomes unresponsive
- Worst case: nested interrupts may be missed

### Experiment B: Reverting to Delay Loop

Replacing timer interrupt with `delay_ms(200)`:

- Busy-wait loop wastes CPU cycles (99%+ idle)
- Alarm may still trigger but has no real-time guarantee
- 100ms deadline NOT guaranteed - depends on loop execution time
- No concurrent processing possible
- System cannot respond to other events during delay

---

## Section 7: Reflection Questions

### Q1: Why are global variables declared as volatile?

Because they are shared between the ISR (producer) and main loop (consumer). Without `volatile`:

- Compiler may cache the variable in a register
- Main loop might read stale/old values
- ISR changes are invisible to main loop code
- Compiler optimization can break the data sharing contract

### Q2: Could we set a flag in ISR and activate alarm in main loop?

**Technically yes, but NOT recommended for safety-critical systems:**

- Flag approach: ISR sets flag, main loop checks and calls `alarm_on()`
- Problem: Main loop may be delayed by other processing
- Alarm latency becomes: ISR latency + main loop scheduling delay
- In this system: could be >100ms (violates NFR-01)
- **Direct alarm in ISR guarantees <1ms latency** - always within 100ms

### Q3: What happens with multiple interrupts of different priorities?

- **Lower-priority task running when timer fires:** Timer (higher priority) preempts it immediately - ISR runs, then returns to lower-priority task
- **Higher-priority interrupt during TIM2's ISR:** TIM2 ISR gets preempted, higher-priority ISR runs, then returns to TIM2 ISR
- STM32 NVIC handles this automatically via priority-based preemption
- **Priority inversion risk:** If a low-priority interrupt blocks a high-priority one, real-time deadlines can be missed (covered in Week 5)

---

## Key Takeaways

1. **ISR-based timing** ensures predictable, deterministic behavior
2. **volatile** is essential for ISR-main loop communication
3. **Safety-critical alarms** must be triggered directly in ISR, not deferred
4. **Busy-wait loops** break real-time guarantees
5. **Priority-based preemption** in NVIC enables multi-interrupt systems
