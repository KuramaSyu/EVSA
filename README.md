# EVSA

## Design Patterns
### Strategy Pattern
Die Idee: Stelle dir eine Klasse für einen Spielcharakter vor. Dieser hat ein Schwert, mit welchem er zuschlagen kann. Der staight-forward Weg währe, eine `Character.swing_sword()` Methode einzubringen. Jetzt wird dir allerdings gesagt, dass er auf einmal auch andere Schwerter Nutzen kann. Wahrscheinlich würdest du den Code jetzt so anpassen, dass man in die `swing_sword()` Methode nun einfach das spezifische Schwert dazugibt, also `Character.swing_sword(Sword)`. Weil das allerdings nicht genug ist, wird die jetzt gesagt, dass er nicht nur unterschiedliche Schwerter nutzen soll, sondern auch Waffen. Dir ist klar, dass `Character.swing_sword(Weapon)` nicht wie vorgesehen funktionieren wird. Also fügst du neben `Character.swing_sword(Sword)` noch `Character.shoot(Weapon)`. Wenn der Spieler jetzt die Maustaste zum Angreifen klickt, musst du also erstmal gucken, ob er jetzt eine Waffe oder ein Schwert in der Hand hat e.g.
```python
class Character:
	current_weapon: AssaultRifle | Sword
	
	def on_attack_click(self):
        if isinstance(self.current_weapon, Sword):
            self.swing_sword(self.current_weapon)
            
        elif isinstance(self.current_weapon, AssaultRifle):
            self.shoot(self.current_weapon)
            
        else:
	        raise RuntimeError("Weapontype unknown")


    def swing_sword(self, sword: Sword): ...
    def shoot(self, weapon: AssaultRifle): ...
```
Schön. Wir haben einen Charakter, der sowohl mit Schusswaffen als auch Schwertern umgehen kann. Nun wird der allerdings gesagt, dass wir auch gerne Bögen hatten, die man unterschiedlich stark spannen kann und Magiewaffen wären auch nicht schlecht. Du fügst also noch `Character.use_magic_rod(Rod)` und `Character.draw_bow(Bow)` ein. Die `on_attack_click()` Methode muss also nun schon zwischen 4 Waffentypen unterscheiden.

An diesem Punkt wäre die saubere Lösung das Strategy Pattern gewesen. Hier hätten wir eine Strategie für jede Waffe einbauen können. Jede Strategie hätte dann eine `Strategy.execute()` Methode, die dafür sorgt, dass die Waffe wie vorgesehen benutzt wird. Die `SwordStrategy` würde das Schwert schwingen, die `AssaultRifleStragety.execute()` würde einen Schuss abgeben usw. Die `Character`Klasse könnte nun ein neues Feld `Character.weapon_strategy: Strategy` bekommen, und sobald der Spieler klickt, müssen wir nur noch `weapon_stragey.execute()` ausführen. Das ganze würde in etwa so aussehen: 
```python
from abc import ABC, abstractmethod

# Jede Strategie soll eine execute() Methode haben
class AttackStrategy(ABC):
    @abstractmethod
    def execute(self) -> None:
        ...

class SwordStrategy(AttackStrategy):
    def execute(self) -> None:
        print("Schwert geschwungen")

class AssaultRifleStrategy(AttackStrategy):
    def execute(self) -> None:
        print("Salve abgefeuert")

class BowStrategy(AttackStrategy):
    def execute(self) -> None:
        print("Pfeil abgeschossen")

class MagicRodStrategy(AttackStrategy):
    def execute(self) -> None:
        print("Zauber ausgelöst")


class Character:
    def __init__(self, weapon_strategy: AttackStrategy):
        self.weapon_strategy = weapon_strategy  # just pass in the strat

    def on_attack_click(self) -> None:
        self.weapon_strategy.execute()  # one line, always works
        
c = Charakter()
sword = SwordStragety()
c.weapon = sword
c.on_attack_click()  # Schwert geschwungen
bow = BowStragety()
c.weapon = bow  
c.on_attack_click()  # Pfeil abgeschossen
```
Formal würde das ganze wie folgt aussehen: 
`Context` wäre der `Character`, `Stragety` wäre `AttackStragety`, und ConcreteStragetyA .. ConcreteStragetyC wären die jeweiligen Waffenimplementierungen. 
Das was wir davon haben, sind während der Laufzeit austauschbare Algorithmen. Ein Lieferservice könnte so Beispielsweise Strategien für die unterschiedlichen Lieferdienste erstellen. Also `DhlStrategy`, `UpsStragety`, `HermesStrategy`, ... eben da alle ein Stück anders funktionieren werden, aber irgendwo schon ähnlich sind.
```mermaid
classDiagram
    class Context {
        -strategy: Strategy
        +setStrategy(Strategy)
        +executeStrategy()
    }

    class Strategy {
        <<interface>>
        +execute(data)
    }

    class ConcreteStrategyA {
        +execute(data)
    }

    class ConcreteStrategyB {
        +execute(data)
    }

    class ConcreteStrategyC {
        +execute(data)
    }

    Context --> Strategy : uses
    Strategy <|.. ConcreteStrategyA : implements
    Strategy <|.. ConcreteStrategyB : implements
    Strategy <|.. ConcreteStrategyC : implements
```


### Observer Pattern
---
_Observer = Beobachter_

Unser halbfertiges Spiel geht in die nächste Runde. Um unique zu sein, haben wir uns dazu entschieden, eine `HealthBar` einzubauen, damit der Spieler sieht, wann er stirbt. Das, was wir auf jeden Fall brauchen, ist ein `Character.health` Attribut und eine `HealthBar` Klasse. Flogend muss diese immer ein Update bekommen, sobald der Spieler Schaden bekommt. Der einfachste Weg hierfür ist, dass wir die HealthBar die zum Spieler gehört, direkt in die Character Klasse rein geben, damit diese folgend `Healthbar.set_health()` Methode aus der `Character` Klasse bei Lebensänderungen aufgerufen werden kann:

```py
class HealthBar:
    def __init__(self) -> None:
        self.health: int = 100

    def set_health(self, value: int) -> None:
        self.health = value
        print(f"HealthBar: {self.health}/100")

class Character:
    # beim Erstellen des Charakters geben wir direkt die jeweilige
    # HealthBar mit dazu, damit die Chracter Klasse 
    # die HealthBar aktuell hält
    def __init__(self, healthbar: HealthBar) -> None:
        self.healthbar = healthbar  
        self.health

    def take_damage(self, amount: int) -> None:
        new_health = self.healthbar.health - amount
        self.health = new_health
        self.healthbar.set_health(new_health)  # keep healthbar updated

    def heal(self, amount: int) -> None:
        new_health = self.healthbar.health + amount
        self.health = new_health
        self.healthbar.set_health(new_health)
```
Jetzt wird also immer, wenn der Spieler Schaden bekommt, `HealthBar.set_healt()` aufgerefuen, damit die HealthBar aktuell bleibt. Nun kommt eine neue Anforderung hinzu, dass wir ein Achievement System einbauen wollen, damit der Spieler ein Achievement bekommt, sobald er mal mit 1HP überlebt. Demach brauchen wir erstmal eine `AchievementSystem` Klasse, welche die Logik enthält,
ob und wann ein Spieler ein Achievement bekommt:
```py
class AchievementSystem:
    def check_for_achievement(self, player: Character) -> None:
        if player.health == 1:
            self.award_1_hp_achievement(player)
    
    def award_1_hp_achievement(self, player: Character) -> None:
        print(f"Achievement unlocked for {player}: Survived with 1 HP!")
```
Aber wie sorgen wir jetzt dafür, dass das `AchievementSystem` die Lebensänderungen des Spielers mitbekommt? Der derzeitige einfachste Ansatz wäre, dass wir neben der `HealthBar` auch noch das `AchievementSystem` in die `Character` Klasse geben, damit bei `Character.take_damage()` und `Character.heal()` auch `AchievementSystem.check_for_achievement()` aufgerufen wird:
```py
class HealthBar:
    def __init__(self) -> None:
        self.health: int = 100

    def set_health(self, value: int) -> None:
        self.health = value
        print(f"HealthBar: {self.health}/100")

class Character:
    def __init__(self, healthbar: HealthBar, achievement_system: AchievementSystem) -> None:
        self.healthbar = healthbar
        self.achievement_system = achievement_system
        self.health = 100

    def take_damage(self, amount: int) -> None:
        new_health = self.healthbar.health - amount
        self.health = new_health
        self.healthbar.set_health(new_health)
        self.achievement_system.check_for_achievement(self)

    def heal(self, amount: int) -> None:
        new_health = self.healthbar.health + amount
        self.health = new_health
        self.healthbar.set_health(new_health)
        self.achievement_system.check_for_achievement(self) 

player_a_healthbar = HealthBar()
achievement_system = AchievementSystem()
player_a = Character(player_a_healthbar, achievement_system)
player_a.take_damage(1)
# healthbar prints: HealthBar: 99/100
player_a.take_damage(98)
# healthbar prints: HealthBar: 1/100
# achievement system prints: Achievement unlocked for <Character object>: Survived with 1 HP
```
Das funktioniert zwar, allerdings sind `HealthBar` und `AchievementSystem` jetzt direkt an die `Character` Klasse gebunden - das ist ein Code Smell.
Das Problem liegt darin, dass mit jeder weiteren Komponente, neben der `HealthBar` und dem `AchievementSystem`, die auf Lebensänderungen reagieren soll, die `Character` Klasse immer größer wird, da wir jede einzelne dieser 
Komponenten in den Konstruktor der `Character` Klasse `Character.__init__()` geben müssen. Außerdem muss die `Character` Klasse bei jeder Lebensänderung (`Character.take_damage()` und `Character.heal()`) jede einzelne Komponente aufrufen, damit diese auf die Lebensänderung reagieren kann. Das ist nicht nur unübersichtlich, sondern auch fehleranfällig, da man leicht vergisst, eine Komponente aufzurufen.

Der saubere Weg wäre ein Observer Pattern. Hierbei würde unser `Character`
eine reihe von Observern entgegennehmen, wobei egal ist, welche Klasse dahinter liegt, solange wie diese eine `Observer.update(player: Character)` Methode besitzt. Das stellen wir mit einem Interface, oder einer Abstracten Base Class in Python sicher: 
```py
from abc import ABC, abstractmethod

class Observer(ABC):
    @abstractmethod
    def update(self, player: Character) -> None:
        ...
```
Diese Abstract Base Class (ABC) kann nun in die `HealthBar` und `AchievementSystem` Klassen eingebaut werden:
```py
class HealthBar(Observer):
    def __init__(self) -> None:
        self.health: int = 100
    
    # die update() Methode des ABCs die wir brauchen
    def update(self, player: Character) -> None:
        self.health = player.health
        print(f"HealthBar: {self.health}/100")

class AchievementSystem(Observer):
    def update(self, player: Character) -> None:
        if player.health == 1:
            self.award_1_hp_achievement(player)
    
    def award_1_hp_achievement(self, player: Character) -> None:
        print(f"Achievement unlocked for {player}: Survived with 1 HP!")
```
Folgend entfernen wir die direkten Bindungen von `HealthBar` und `AchievementSystem` aud dem Konstruktor der `Character` Klasse und fügen 
stattdessen eine Liste von Observern ein, welche wir bei Lebensänderungen benachrichtigen:
```py
class Character:
    def __init__(self) -> None:
        self.health = 100
        self.observers: list[Observer] = []

    def attach(self, observer: Observer) -> None:
        """Einen Observer hinzufügen"""
        self.observers.append(observer)

    def detach(self, observer: Observer) -> None:
        """Einen Observer entfernen"""
        self.observers.remove(observer)

    def notify(self) -> None:
        """Alle Observer benachrichtigen"""
        for observer in self.observers:
            observer.update(self)  # wir übergeben uns selbst an den Observer

    def take_damage(self, amount: int) -> None:
        new_health = self.health - amount
        self.health = new_health
        self.notify()

    def heal(self, amount: int) -> None:
        new_health = self.health + amount
        self.health = new_health
        self.notify()

achievement_system = AchievementSystem()
player_a_healthbar = HealthBar()
player_a = Character()

# nun binden wir alle Klassen, welche Interesse am Spieler haben, 
# an den Spieler an, damit diese bei Lebensänderungen benachrichtigt werden
player_a.attach(player_a_healthbar)
player_a.attach(achievement_system)
player_a.take_damage(1)
# healthbar prints: HealthBar: 99/100
player_a.take_damage(98)
# healthbar prints: HealthBar: 1/100
# achievement system prints: Achievement unlocked for <Character object>: Survived with 1 HP
player.detach(achievement_system)  # der Spieler hat das Achivement bekommen, daher können wir es auch entfernen, damit der Spieler das Achivement nicht nochmal bekommt
player_a.heal(1)
# healthbar prints: HealthBar: 2/100
player_a.take_damage(1)
# healthbar prints: HealthBar: 1/100
# no achievement system prints, since we detached it
```
Formal würde das ganze wie folgt aussehen:

Formal | im Beispiel
---|---
`Subject` | `Character`
`ConcreteSubject` | `Character` (wir haben keine Abstrakte Klasse/Interface für den Spieler, um das Beispiel einfach zu halten. Wenn wir jedoch Character mit weniger, mehr oder unendlich Leben haben, wäre es sinnvoll, eine Abstrakte Klasse/Interface für den Spieler zu erstellen, und jeweils eine Unterklasse für die Spezialisierungen wie `HalfHPCharacter`, `FullHPCharacter`, `InfiniteHPCharacter` zu erstellen)
`ConcreteSubject.setState()` | `Character.take_damage()` und `Character.heal()`
`Observer` | `Observer` (ABC)
`ConcreteObserverA` | `HealthBar`
`ConcreteObserverB` | `AchievementSystem`
```mermaid
classDiagram
    class Subject {
        -observers: list~Observer~
        +attach(observer)
        +detach(observer)
        +notify()
    }

    class ConcreteSubject {
        -state
        +getState()
        +setState(state)
    }

    class Observer {
        <<interface>>
        +update(subject)
    }

    class ConcreteObserverA {
        -observer_state
        +update(subject)
    }

    class ConcreteObserverB {
        -observer_state
        +update(subject)
    }

    Subject <|-- ConcreteSubject
    Observer <|.. ConcreteObserverA : implements
    Observer <|.. ConcreteObserverB : implements
```

