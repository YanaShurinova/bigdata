# Лабораторная 3

1. RideCleansing

Изменения:
    public boolean filter(TaxiRide taxiRide) {
                return GeoUtils.isInNYC(taxiRide.startLon, taxiRide.startLat)
                        && GeoUtils.isInNYC(taxiRide.endLon, taxiRide.endLat);
            }

Описание:
    Метод, проверяющий, что координаты начала поездки и координаты конца поездки соответствуют Нью-Йорку.

Тесты:
![RideCleansingTest.jpg](https://github.com/YanaShurinova/bigdata/lab3/RideCleansingTest.jpg)

2. RideAndFares

Изменения:
    public static class EnrichmentFunction
            extends RichCoFlatMapFunction<TaxiRide, TaxiFare, RideAndFare> {
        private ValueState<TaxiRide> rideState;
        private ValueState<TaxiFare> fareState;
        @Override
        public void open(Configuration config) {
            rideState = getRuntimeContext().getState(new ValueStateDescriptor<>("saved ride", TaxiRide.class));
            fareState = getRuntimeContext().getState(new ValueStateDescriptor<>("saved fare", TaxiFare.class));
        }

        @Override
        public void flatMap1(TaxiRide ride, Collector<RideAndFare> out) throws Exception {

            TaxiFare fare = fareState.value();
            if (fare != null) {
                fareState.clear();
                out.collect(new RideAndFare(ride, fare));
            } else {
                rideState.update(ride);
            }
        }

        @Override
        public void flatMap2(TaxiFare fare, Collector<RideAndFare> out) throws Exception {

            TaxiRide ride = rideState.value();
            if (ride != null) {
                rideState.clear();
                out.collect(new RideAndFare(ride, fare));
            } else {
                fareState.update(fare);
            }
        }
    }

Описание:
    open - получаем taxiride и taxifare
    flatMap1 и flatMap2 - обрабатываем поездку/событие: если первый раз обрабатывается - сохраняет поездку/событие;
    если не в первый раз - объединяются в складываются в поток

Тесты:
![RideAndFaresTest.jpg](https://github.com/YanaShurinova/bigdata/lab3/RideAndFaresTest.jpg)

3. HourlyTips

Изменения:
    public JobExecutionResult execute() throws Exception {

        // set up streaming execution environment
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        // start the data generator and arrange for watermarking
        DataStream<TaxiFare> fares = env.addSource(source);



        /* You should explore how this alternative (commented out below) behaves.
         * In what ways is the same as, and different from, the solution above (using a windowAll)?
         */

        // DataStream<Tuple3<Long, Long, Float>> hourlyMax = hourlyTips.keyBy(t -> t.f0).maxBy(2);
        DataStream<Tuple3<Long, Long, Float>> hourlyMax = fares
                .assignTimestampsAndWatermarks(WatermarkStrategy.<TaxiFare>forMonotonousTimestamps()
                        .withTimestampAssigner((fare, t) -> fare.getEventTimeMillis()))
                .keyBy(fare -> fare.driverId)
                .window(TumblingEventTimeWindows.of(Time.hours(1)))
                .process(new AddTips())
                .windowAll(TumblingEventTimeWindows.of(Time.hours(1))).maxBy(2);

        hourlyMax.addSink(sink);

        // execute the transformation pipeline
        return env.execute("Hourly Tips");
    }
    public static class AddTips
            extends ProcessWindowFunction<TaxiFare, Tuple3<Long, Long, Float>, Long, TimeWindow> {

        @Override
        public void process(
                Long key,
                Context context,
                Iterable<TaxiFare> fares,
                Collector<Tuple3<Long, Long, Float>> out) {

            float sumOfTips = 0F;
            for (TaxiFare f : fares) {
                sumOfTips += f.tip;
            }
            out.collect(Tuple3.of(context.window().getEnd(), key, sumOfTips));
        }
    }

Описание:
    Потоку с оплатой поездок назначаются метки, зависящие от даты совершения оплаты, для последующего функционирования оконных функций.
    Далее объектам назначаются ключи (keyBy(fare -> fare.driverId)). После этого поток разбивается на временные окна, длительность которых - один час и к каждому ключу применяется функция, которая суммирует чаевые. Затем применяется еще одно деление на временные окна, длительностью час(без учета ключа) и затем выбирается водитель с наибольшими чаевыми.

Тесты:
![HourlyTipsTest.jpg](https://github.com/YanaShurinova/bigdata/lab3/HourlyTipsTest.jpg)

4. LongRides

Изменения:
public static class AlertFunction extends KeyedProcessFunction<Long, TaxiRide, Long> {
    private ValueState<TaxiRide> rideState;
    @Override
    public void open(Configuration config) throws Exception {
        ValueStateDescriptor<TaxiRide> rideStateDescriptor =
                new ValueStateDescriptor<>("ride event", TaxiRide.class);
        rideState = getRuntimeContext().getState(rideStateDescriptor);
    }

    @Override
    public void processElement(TaxiRide ride, Context context, Collector<Long> out)
            throws Exception {
        TaxiRide firstRideEvent = rideState.value();
        long timerTime;
        int duration;
        try{
            if (firstRideEvent == null) {
                // whatever event comes first, remember it
                rideState.update(ride);

                if (ride.isStart) {
                    // we will use this timer to check for rides that have gone on too long and may
                    // not yet have an END event (or the END event could be missing)
                    timerTime = ride.eventTime.plusSeconds(120 * 60).toEpochMilli();
                    context.timerService().registerEventTimeTimer(timerTime);
                }
            } else {
                if (ride.isStart) {
                    duration = Duration.between(ride.eventTime, firstRideEvent.eventTime)
                            .compareTo(Duration.ofHours(2));
                    if (duration>0) {
                        out.collect(ride.rideId);
                    }
                } else {
                    // the first ride was a START event, so there is a timer unless it has fired
                    timerTime = firstRideEvent.eventTime.plusSeconds(120 * 60).toEpochMilli();
                    context.timerService().deleteEventTimeTimer(timerTime);
                    duration = Duration.between(firstRideEvent.eventTime, ride.eventTime)
                            .compareTo(Duration.ofHours(2));
                    // perhaps the ride has gone on too long, but the timer didn't fire yet
                    if (duration>0) {
                        out.collect(ride.rideId);
                    }
                }

                // both events have now been seen, we can clear the state
                // this solution can leak state if an event is missing
                // see DISCUSSION.md for more information
                rideState.clear();
            }
        }catch(Exception e){
            throw new Exception();
        }
    }

    @Override
    public void onTimer(long timestamp, OnTimerContext context, Collector<Long> out)
            throws Exception {
        // the timer only fires if the ride was too long
        out.collect(rideState.value().rideId);

        // clearing now prevents duplicate alerts, but will leak state if the END arrives
        rideState.clear();
    }
}

Описание:

Если объект функции первый полученный – он его сохраняет, если событие – начало поездки: в поток кладется слишком долгая поездка.
Если объект функции не первый полученный - проверяется, соответствует ли данная поездка минимальной длительности для предупреждения и добавляет в выходной поток, если лимит по времени был превышен.
Во всех случаях состояние очищается.

Тесты:
![LongRidesTest.jpg](https://github.com/YanaShurinova/bigdata/lab3/LongRidesTest.jpg)