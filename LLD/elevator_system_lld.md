# Elevator System — Low Level Design

## 1. Clarify Requirements (say this out loud in the interview)

**Functional**
- N elevators serving M floors in a building.
- A user on a floor presses UP/DOWN (external request).
- A user inside an elevator presses a destination floor button (internal request).
- System picks the "best" elevator to serve an external request.
- Each elevator has doors that open/close, and moves one floor at a time.
- Display shows current floor + direction inside and outside the elevator.

**Non-functional**
- Extensible scheduling algorithm (nearest-car, SCAN/LOOK, zoning, etc.)
- Thread-safe (multiple requests can arrive concurrently).
- Easy to add elevators, floors, or features (e.g., maintenance mode, capacity limits) later.

**Assumptions to state**
- Single building, real-time dispatch, no distributed system concerns.
- Capacity limits and priority (e.g., fire mode) are "nice to have" extensions, not core.

## 2. Identify Core Entities

| Entity | Responsibility |
|---|---|
| `Elevator` (Car) | Owns its own state, current floor, direction, and pending stops |
| `ElevatorController` (Dispatcher) | Receives requests, chooses an elevator, delegates the move |
| `Request` | Represents an external (hall call) or internal (car call) request |
| `Direction` | UP / DOWN / IDLE |
| `ElevatorState` | IDLE / MOVING / DOOR_OPEN / MAINTENANCE (State pattern) |
| `SchedulingStrategy` | Pluggable algorithm to pick the best elevator (Strategy pattern) |
| `Display` / `Button` | Observers notified of elevator state changes |

## 3. Design Patterns Used (call these out explicitly — interviewers love this)

- **State Pattern** — elevator behavior (can it accept a request? should doors open?) changes based on state (`Idle`, `Moving`, `DoorOpen`, `Maintenance`).
- **Strategy Pattern** — the dispatcher's "which elevator should serve this request" logic is swappable (nearest car, SCAN, look-ahead, load-balanced).
- **Observer Pattern** — floor displays / car displays subscribe to elevator state changes instead of being polled.
- **Singleton** — `ElevatorController` (one dispatcher per building) — mention it but note in real systems you'd inject it, not hard-code a singleton.

## 4. Class Diagram (textual)

```
ElevatorController (Singleton)
 ├── List<Elevator>
 ├── SchedulingStrategy strategy
 ├── submitExternalRequest(floor, direction)
 └── submitInternalRequest(elevatorId, destFloor)

Elevator
 ├── id, currentFloor, capacity
 ├── ElevatorState state
 ├── Direction direction
 ├── TreeSet<Integer> upStops, downStops
 ├── List<Observer> observers
 ├── step()          // move one floor / open door
 ├── addStop(floor)
 └── notifyObservers()

ElevatorState (interface)
 ├── IdleState
 ├── MovingState
 ├── DoorOpenState
 └── MaintenanceState

SchedulingStrategy (interface)
 └── NearestCarStrategy implements SchedulingStrategy

Request
 ├── ExternalRequest(floor, direction)
 └── InternalRequest(destFloor)

Observer (interface)
 └── Display implements Observer
```

## 5. Key Class Interactions (flow)

1. User presses hall button → `ElevatorController.submitExternalRequest(floor, dir)`
2. Controller asks `SchedulingStrategy` to pick the best `Elevator` from its list.
3. Chosen elevator adds the floor to its stop queue (`upStops`/`downStops` — sorted sets so SCAN order falls out naturally).
4. A scheduler thread/timer calls `elevator.step()` periodically:
   - if at a stop → transition to `DoorOpenState`, notify observers, remove stop
   - else → move one floor toward next stop, transition to `MovingState`
5. When user inside presses a floor → `Elevator.addStop(destFloor)` directly (internal request, no dispatch needed).
6. State transitions and floor changes notify `Display` observers.

## 6. Java Implementation

```java
// ---------- Enums ----------
public enum Direction { UP, DOWN, IDLE }

// ---------- Request ----------
public abstract class Request {
    protected final int floor;
    protected Request(int floor) { this.floor = floor; }
    public int getFloor() { return floor; }
}

public class ExternalRequest extends Request {
    private final Direction direction;
    public ExternalRequest(int floor, Direction direction) {
        super(floor);
        this.direction = direction;
    }
    public Direction getDirection() { return direction; }
}

public class InternalRequest extends Request {
    public InternalRequest(int destFloor) { super(destFloor); }
}

// ---------- Observer ----------
public interface ElevatorObserver {
    void onStateChange(Elevator elevator);
}

public class Display implements ElevatorObserver {
    @Override
    public void onStateChange(Elevator elevator) {
        System.out.printf("Elevator %d -> floor %d, dir %s, state %s%n",
            elevator.getId(), elevator.getCurrentFloor(),
            elevator.getDirection(), elevator.getState().name());
    }
}

// ---------- State Pattern ----------
public interface ElevatorState {
    void handle(Elevator elevator);
    String name();
}

public class IdleState implements ElevatorState {
    public void handle(Elevator elevator) {
        if (!elevator.hasPendingStops()) return;
        elevator.setState(new MovingState());
    }
    public String name() { return "IDLE"; }
}

public class MovingState implements ElevatorState {
    public void handle(Elevator elevator) {
        int target = elevator.peekNextStop();
        int current = elevator.getCurrentFloor();

        if (target == current) {
            elevator.setState(new DoorOpenState());
            return;
        }
        int next = target > current ? current + 1 : current - 1;
        elevator.setCurrentFloor(next);
        elevator.setDirection(target > current ? Direction.UP : Direction.DOWN);
    }
    public String name() { return "MOVING"; }
}

public class DoorOpenState implements ElevatorState {
    public void handle(Elevator elevator) {
        elevator.removeCurrentStop();
        // door closes after handling in a real system; simplified here
        elevator.setState(elevator.hasPendingStops() ? new MovingState() : new IdleState());
        if (!elevator.hasPendingStops()) elevator.setDirection(Direction.IDLE);
    }
    public String name() { return "DOOR_OPEN"; }
}

public class MaintenanceState implements ElevatorState {
    public void handle(Elevator elevator) { /* no-op: out of service */ }
    public String name() { return "MAINTENANCE"; }
}

// ---------- Elevator ----------
import java.util.*;

public class Elevator {
    private final int id;
    private int currentFloor = 0;
    private Direction direction = Direction.IDLE;
    private ElevatorState state = new IdleState();

    private final TreeSet<Integer> upStops = new TreeSet<>();
    private final TreeSet<Integer> downStops = new TreeSet<>(Collections.reverseOrder());
    private final List<ElevatorObserver> observers = new ArrayList<>();

    public Elevator(int id) { this.id = id; }

    public void addStop(int floor) {
        if (floor > currentFloor) upStops.add(floor);
        else if (floor < currentFloor) downStops.add(floor);
        state.handle(this); // wake up from idle if needed
        notifyObservers();
    }

    public void step() {
        state.handle(this);
        notifyObservers();
    }

    public boolean hasPendingStops() { return !upStops.isEmpty() || !downStops.isEmpty(); }

    public int peekNextStop() {
        if (direction == Direction.DOWN && !downStops.isEmpty()) return downStops.first();
        if (!upStops.isEmpty()) return upStops.first();
        if (!downStops.isEmpty()) return downStops.first();
        return currentFloor;
    }

    public void removeCurrentStop() {
        upStops.remove(currentFloor);
        downStops.remove(currentFloor);
    }

    public void subscribe(ElevatorObserver o) { observers.add(o); }
    private void notifyObservers() { observers.forEach(o -> o.onStateChange(this)); }

    // getters/setters
    public int getId() { return id; }
    public int getCurrentFloor() { return currentFloor; }
    public void setCurrentFloor(int f) { currentFloor = f; }
    public Direction getDirection() { return direction; }
    public void setDirection(Direction d) { direction = d; }
    public ElevatorState getState() { return state; }
    public void setState(ElevatorState s) { state = s; }
}

// ---------- Strategy Pattern ----------
public interface SchedulingStrategy {
    Elevator selectElevator(List<Elevator> elevators, ExternalRequest request);
}

public class NearestCarStrategy implements SchedulingStrategy {
    @Override
    public Elevator selectElevator(List<Elevator> elevators, ExternalRequest request) {
        Elevator best = null;
        int bestDist = Integer.MAX_VALUE;
        for (Elevator e : elevators) {
            if (e.getState() instanceof MaintenanceState) continue;
            int dist = Math.abs(e.getCurrentFloor() - request.getFloor());
            // prefer elevators already moving in the requested direction (simplified scoring)
            boolean sameDirOrIdle = e.getDirection() == Direction.IDLE
                || e.getDirection() == request.getDirection();
            if (sameDirOrIdle && dist < bestDist) {
                bestDist = dist;
                best = e;
            }
        }
        return best != null ? best : elevators.get(0); // fallback
    }
}

// ---------- Controller (Singleton + Facade) ----------
public class ElevatorController {
    private static final ElevatorController INSTANCE = new ElevatorController();
    private final List<Elevator> elevators = new ArrayList<>();
    private SchedulingStrategy strategy = new NearestCarStrategy();

    private ElevatorController() {}
    public static ElevatorController getInstance() { return INSTANCE; }

    public void registerElevator(Elevator e) { elevators.add(e); }
    public void setStrategy(SchedulingStrategy s) { this.strategy = s; }

    public void submitExternalRequest(ExternalRequest request) {
        Elevator chosen = strategy.selectElevator(elevators, request);
        chosen.addStop(request.getFloor());
    }

    public void submitInternalRequest(int elevatorId, InternalRequest request) {
        elevators.stream()
            .filter(e -> e.getId() == elevatorId)
            .findFirst()
            .ifPresent(e -> e.addStop(request.getFloor()));
    }

    public void tick() { elevators.forEach(Elevator::step); }
}

// ---------- Demo ----------
public class Main {
    public static void main(String[] args) {
        ElevatorController controller = ElevatorController.getInstance();

        for (int i = 1; i <= 3; i++) {
            Elevator e = new Elevator(i);
            e.subscribe(new Display());
            controller.registerElevator(e);
        }

        controller.submitExternalRequest(new ExternalRequest(5, Direction.UP));
        controller.submitInternalRequest(1, new InternalRequest(9));

        for (int i = 0; i < 10; i++) controller.tick();
    }
}
```

## 7. Extensions to Mention If Asked

- **Capacity/weight limit** → check before accepting internal requests, reject or queue.
- **SCAN/LOOK optimization** → already partially modeled via `TreeSet` ordering; extend `MovingState` to keep serving stops in the current direction before reversing.
- **Zoning for peak traffic** → assign elevators to floor ranges during rush hour (swap `SchedulingStrategy`).
- **Fault tolerance** → `MaintenanceState` already isolates a broken car from dispatch.
- **Concurrency** → wrap `Elevator` mutations in locks, or make the controller single-threaded via an event queue (actor-style) to avoid race conditions on `upStops`/`downStops`.
- **Persistence/logging** → observers can log to a DB instead of `System.out`.

## 8. How to Pace This in a 45-min Interview

1. **5 min** — requirements + assumptions (out loud, get interviewer buy-in).
2. **10 min** — entities + class diagram on whiteboard.
3. **5 min** — call out patterns (State, Strategy, Observer) and *why* each fits.
4. **20 min** — code the core: `Elevator`, `State` classes, `Controller`, one `Strategy`.
5. **5 min** — extensions/trade-offs discussion (this is often what differentiates strong candidates).
