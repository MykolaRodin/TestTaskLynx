# Test task description

## Дано:
1. є житлова будівля висотою F поверхів
2. будівля має 2 ліфти які зупиняються на всіх поверхах
3. кожний поверх має кнопки виклику ліфтів (одну кнопку для обох)
4. кожний ліфт має клавіатуру з кнопками всіх поверхів, і кнопкою термінової зупинки (STOP)

## Завдання:
1. обгрунтовано запропонуйте математичну абстракцію для опису моделі даної системи
2. складіть список характеристик за якими робота даної системи могла б вважатись її користувачами задовільною
3. надайте список критеріїв за якими, на вашу думку, користувачі оцінюють якість роботи такої системи
4. запропонуйте оптимальні алгоритми уявної прошивки контролерів такої системи, себто:
* контролера кнопок виклику на поверхах
* контролера кнопок в ліфті (логіка має бути однаковою для обох ліфтів)
надайте Ваші обгрунтування оптимальності наведенних алгоритмів

У тому разі якщо завдання вдається Вам надмірно складним у частині опису алгоритмів схематично опишіть як би Ви складали такі алгоритми в разі наявності більшого часу

Використання ChatGPT або інших LLMs не забороняється (і навіть вітається у тому разі, якщо Ви своїми словами можете обгрунтувати їхнє використання), але у разі їхнього використання ми хотіли б бачити логи відповідних чатів.

# Approach details
## 1. The abstraction of the model might look like this:
 1.1. The elevator can be described as struct that contains:

  1.1.1. The vector of booleans of size F that corresponds to elevator buttons (If the elevator stands on some floor, corresponding index is set to true. If elevator is moving, there might be several trues in the vector if several people have entered the elevator and are moving to different floors);

  1.1.2. The vector of special elevator buttons of size 1 that contains just STOP button (It might be tempting to optimize it and use just one vector where STOP button is plcaed at zero index. Yet this premature optimization might fail if there is an underground floor or further modifications might require to use extra speial buttons (for example, a button that calls a reparement operator from inside the elevator if it suddenly stops functioning)).

  1.1.3. The state machine that corresponds to one of possible elevator states:
    - STAND (When the elevator stands at some floor);
    - OVERLOAD (When the elevator stands at some floor yet the weight of the passengers is higher then allowed);
    - MOVEMENT_TO_USER (When the elevator is moving between the floors on user's request);
    - MOVEMENT_WITH_PASSENGERS (When the elevator is moving between the floors with users);
    - STAND_BETWEEN_MOVEMENTS (If several people have entered the elevator and are moving to different floors, the elevator might stop at some floor and then continue the movement. If the elevator has just arrived to the user, it uses this state as well);
    - OUT_OF_ORDER (If the elevator is out of order).

  1.1.4. The indicator to indicate which floor the elevator is currently at.

  1.1.5. The indicator that indicates the elevator is overloaded.

 1.2. The floor equipment can be described as a struct that contains:
  - floor button.

 1.3. The controller of elevator buttons can be described as struct that contains (please see p.4 for extra details):
  - the queue to receive requests from the controller of floor buttons;
  - the reference to the queue to send the notifications to floor indicators;
  - vector of elevators of size 2.

 1.4. The controller of floor buttons can be described as a struct that contains (please see p.4 for extra details):
  - the queue to received the notifications from the elevator to floor indicators;
  - the reference to the queue to send requests to the controller of elevator buttons;
  - vector of floor equipments of size F;
  - vector of floor indicator of size 2 to indicate which floors the elevators are currently are (the indecators on all the floors show the same values).

 1.5. The building can be described as struct that contains:
  - the controller of elevator buttons;
  - the controller of floor buttons.

## 2. Here the list of characteristics is:
 - the speed of the elevator (if the building is high, it might be required to have a fast elevator);
 - the pace of increasing / descreasing the speed (if the pace is to high, some people might endure something like a flight decease);
 - every button (for both elevator buttons and floor buttons) should have a LED inside. The LED should start lightning after user presses the button to indicate the user's request has been received;
 - there should be 2 indicators for both elevators that show the state fo the elevators and the floors the elevator are on (if users see that both elevators are busy, they might change their minds and use stairs instead);
 - there might be a necessity to block some floor buttons (for example, if these floors are under repairement or all the users from the floor reject to pay for the elevator);
 - there should be some defense to prevent the elevator from falling down if the elevator is out if order;
 - there should be a chance for a repairement operator to open the doors manually and let the passengers leave if the elevator suddenly stops operating while moving with pasengers;
 - there should be allowed to pick up other users while the elevator is moving to utilize the elevator to the full extent.


## 3. Here the list of parameter is:
 - the time to wait for an elevator;
 - the time required to get to the required floor;
 - the convenience of movement inside the elevator;
 - the chance for sightseeing through transparent elevator walls in case the time spent in the elevator is too long;
 - the reliability of defence system in case the elevator suddenly stops operating.


## 4. The logic of controllers
 Here the floor button controller logic is:
  - if user presses a floor button, it starts lightning and the request is placed in the elevator queue;
  - when the elevator starts moving, the lightning in the button in the floor stops;
  - while the elevators are moving or stand, the indicator on the floor shows the actual floors the elevators are at reading the notificagtions from the floor queue;

 Here the elevator button controller logic is:
  - if the elevator is out of order (OUT_OF_ORDER state), the corresponding indicators on every floor are shut down or show red color.
  - if the sum of passenger's weights exceeds some limit, the elevator does not start moving (OVERLOAD state) until some passenger leaves the elevator and the weight decreases;
  - if both elevators are moving and the request in the elevator queue is placed, the controller does not read the request from the queue until at least one of elevators does not have STAND state;
  - if at least one of the elevators is in STAND state, the request is read from the queue and the elevator starts moving to the user changing state to MOVING;
  - if both elevators are in STAND state, the one, that is closer to the user should be chosen to decrease the power consumption;
  - if both elevators are on the same distance, the elevator that is higher should be moved down because moving down is supposed to have less power consumption;
  - while elevator is moving to the user (MOVEMENT_TO_USER state), it does not pick up other users;
  - while the elevator is moving after the first user has entered it and became a passenger (MOVEMENT_WITH_USERS state), it is allowed to pick up other users;
  - if some users are picked up, they press the buttons corresponding to the floors they require;
  - the elevator contuniues moving till the time the last passenger leaves and no new passanger comes;
  - if the elevator is in MOVEMENT_WITH_PASSENGERS state and it archives some previously chosen floor, it stops letting some passengers to leave and temporarily passes to STAND_BETWEEN_MOVEMENTS state;
  - if the elevator leaves STAND_BETWEEN_MOVEMENTS state and continues moving, it passes back to MOVEMENT_WITH_PASSENGERS state;
  - when all the passengers have left the elevator, it moved to the STAND state.

P.S. No ChatGPT was used for the approach.
