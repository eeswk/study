#### ordianl 인덱싱 대신 EnumMap을 사용하라.

식물들 배열 하나로 관리하고 생애주기별로 묶는 예제

```java
package effetive.item37;
import java.util.*;

import static java.util.stream.Collectors.groupingBy;
import static java.util.stream.Collectors.toSet;

public class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

    final String name;
    final LifeCycle lifeCycle;

    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }


    public static void main(String[] args) {

        Plant[] garden = {
                new Plant("완두", Plant.LifeCycle.ANNUAL),
                new Plant("옥수수", Plant.LifeCycle.ANNUAL),
                //new Plant("딸기", Plant.LifeCycle.PERENNIAL),
                //new Plant("토마토", Plant.LifeCycle.PERENNIAL),
                new Plant("접시꽃", Plant.LifeCycle.BIENNIAL),
                new Plant("파슬리", Plant.LifeCycle.BIENNIAL)
        };

        // use 1: use EnumMap
        Map<Plant.LifeCycle, Set<Plant>> plantByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);

        for (Plant.LifeCycle lc : Plant.LifeCycle.values())
            plantByLifeCycle.put(lc, new HashSet<>());
        for (Plant p : garden)
            plantByLifeCycle.get(p.lifeCycle).add(p);
        System.out.println(plantByLifeCycle);

        // use 2: not EnumMap and Stream
        // stream use : not exist item, not put value
        //Map<Plant.LifeCycle, List<Plant>> mapList = Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle));
        System.out.println(Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle)));

        // use 3: use EnumMap and Stream
        // stream use : not exist item, not put value
        //EnumMap<Plant.LifeCycle, Set<Plant>> enumMap = Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle, () -> new EnumMap<>(Plant.LifeCycle.class), toSet()));
        System.out.println(Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle, () -> new EnumMap<>(Plant.LifeCycle.class), toSet())));
    }
}

```
결과
```
{ANNUAL=[완두, 옥수수], PERENNIAL=[], BIENNIAL=[접시꽃, 파슬리]}
{ANNUAL=[완두, 옥수수], BIENNIAL=[접시꽃, 파슬리]}
{ANNUAL=[완두, 옥수수], BIENNIAL=[접시꽃, 파슬리]}
```
중첩 EnumMap으로 데이터와 열거 타입 쌍을 연결
```java
package effetive.item37;

import java.util.EnumMap;
import java.util.Map;
import java.util.stream.Collectors;
import java.util.stream.Stream;

import static java.util.stream.Collectors.groupingBy;
import static java.util.stream.Collectors.toMap;

public enum  Phase {
    SOLID, LIQUID, GAS, PLASMA;

    public enum Transition {
        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, LIQUID),
        SUBLITME(SOLID, GAS),
        DEPOSIT(GAS, SOLID),
        //추가
        IONIZE(GAS, PLASMA),
        DEIONIZE(PLASMA, GAS);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        private static final Map<Phase, Map<Phase, Transition>> m
                = Stream.of(values())
                    .collect(groupingBy(t -> t.from, ()-> new EnumMap<>(Phase.class),
                            toMap(t -> t.to, // key
                                    t-> t,   // value
                                    (x,y) -> y, // merge function
                                    () -> new EnumMap<>(Phase.class))));

        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }

    public static void main(String[] args) {
        System.out.println(Transition.m);

        for (Phase src : Phase.values()) {
            for (Phase dst : Phase.values()) {
                Transition transition = Transition.from(src, dst);
                if (transition != null)
                    System.out.printf("%s => %s : %s, %n", src, dst, transition);
            }
        }
    }
}

```
결과
```
{SOLID={LIQUID=MELT, GAS=SUBLITME}, LIQUID={SOLID=FREEZE, GAS=BOIL}, GAS={SOLID=DEPOSIT, LIQUID=CONDENSE, PLASMA=IONIZE}, PLASMA={GAS=DEIONIZE}}
SOLID => LIQUID : MELT,
SOLID => GAS : SUBLITME,
LIQUID => SOLID : FREEZE,
LIQUID => GAS : BOIL,
GAS => SOLID : DEPOSIT,
GAS => LIQUID : CONDENSE,
GAS => PLASMA : IONIZE,
PLASMA => GAS : DEIONIZE,
```
실제 내부에서는 맵들의 맵이 배열들의 배열로 구현되니 낭비되는 공간과 시간도 거의 없이 명확하고 안전하고 유지보수하기 좋다.


_핵심정리_
> - 배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 일반적으로 좋지 않으니, EnumMap을 사용하라.
> - 다차원 관계는 EnumMap<..., EnumMap<...>>으로 표현하라.

[참조] (이펙티브 자바 3판)
