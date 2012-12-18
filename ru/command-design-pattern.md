[On JavaTalks](http://articles.javatalks.ru/articles/3)

Рассмотрим для начала небольшой пример. 
Допустим мы разрабатываем класс, который будет включать/выключать лампу, класс Switch:
```java
public class Switch {
    private Light light;
    public Switcher(Light light){
        this.light = light;
    }
    public void flipUp(){
        light.turnOn();
    }
    public void flipDown(){
        light.turnOff();
    }
}
```

Обычный класс, которому в конструктор передают объект light, его мы включаем в одном методе, выключаем в другом. Но тут произошло следующее: наше приложение стало очень популярным и мы получили заказ сделать то же самое, но теперь с телевизором. Мы конечно хотим переиспользовать старый код, дабы не придумывать все сначала. Поэтому нам приходится лезть в старый код и его модифицировать:
```
public class Switcher {
    private TV tv;
    public Switcher(TV tv){
        this.tv = tv;
    }
    public void tvOn(){
        tv.on();
    }
    public void tvOff(){
        tv.off();
    }
}
```
Однако один из принципов ОО Проектирования гласит, что класс должен быть открыть для расширения, однако закрыт для изменений. В данном случае мы нарушаем этот принцип: каждый раз, если измениться прибор, который нужно включать или выключать, нам нужно лезть в уже готовый класс, изменять его, внося в него новые ошибки.. 
Здесь и приходит на помощь шаблон проектирования Команда. Во всех реализациях этого шаблона есть абстракный базовый класс(или интерфейс). Его реализуют все конкретные команды. Вот он, этот интерфейс:
```
public interface Command {
    public void execute();
}
```
Приведу некоторые классы-команды:
```
public class LightOffCommand implements Command {
    private Light light;

    public LightOffCommand(Light light) {
        this.light = light;
    }

    public void execute() {
        light.turnOff();
    }
}
```
```
public class TvOnCommand implements Command {
    private TV tv;

    public TvOnCommand(TV tv) {
        this.tv = tv;
    }

    public void execute() {
        tv.on();
    }
}
```
Теперь у нас есть классы, которые могут сами включать-выключать то ли телевизор, то ли лампу. Наш старый Switch теперь выглядит так:
```
public class Switch {
    private Command onCommand;
    private Command offCommand;

    public Switch(Command onCommand, Command offCommand) {
         this.onCommand = onCommand;
         this.offCommand = offCommand;
    }

    public void on() {
         onCommand.execute();
    }

    public void off() {
         offCommand.execute();
    }
}
```
Как видите, теперь в конструктор ему передаются команды, которые сами выполняют все действия, Switch теперь общается только с ними и говорит когда _им_ нужно включить/выключить какой-то прибор. Таким образом этот класс ничего не знает ни о каких приборах, теперь единственное, что нам нужно будет сделать, чтоб он заработал с другим прибором, - это создать новые команды для нового прибора и передать их Switch'у. Все, больше не нужно изменять старую, работающую реализацию, достаточно просто добавить новую. Наш код больше не нарушает Opend/Closed Principle. Это и есть шаблон проектирования Команда.
Типичный пример для Команды - это устройство ресторана:
Покупатель(Client) делает заказ(command object), передает его Официантке(invoker), та, в свою очередь, идет к повору, клеет у него где-нибудь на стенке, этот заказ, Повар(receiver) потом видит заказ и делает еду. В данном случае, как видите, Официантка ничего не знает о Поваре, ее дело - отнести заказ куда надо. В этом и есть соль Команды: он отделяет объект, который вызывает действие от объекта, который производит это действие. Кстати, Client, Invoker, Command Object & Reciever - это стандартные имена действующих лиц в этом шаблоне.

Теперь поговорим о том, как осуществить undo-операции с помощью Команды. На самом деле это тривиальный процесс, нам всего лишь нужно сделать действие, противоположное прямому. Для начала добавим метод undo() в базовый интерфейс:
```
public interface Command {
    public void execute();
	public void undo();
}
```
Теперь реализуем его для LightOffCommand:
```
public class LightOffCommand implements Command {
    private Light light;

    public LightOffCommand(Light light) {
        this.light = light;
    }

    public void execute() {
        light.turnOff();
    }
	
	public void undo(){
		light.turnOn();
	}
}
```
Как видите, ничего сложного! Конечно, наш пример уж слишком простой, однако для осуществления undo-операции хватает, как правило:
1. Сохранения предыдущего состояния объекта. Однако это может быть довольно-таки ресурсо-емкой операцией, например, если мы создаем графический редактор, мы не можем сохранять всю графику после каждого действия.
2. Выполнение алгоритма, противоположного прямому. То есть если произошла арифметическая операция +10, то нам всего лишь нужно запомнить число 10 и во время undo сделать -10. 

**Макросы**
Макрос - это какой-то набор команд, который можно повторять или отменять. В том же MS Word можно создать макрос, который будет копировать какой-то текст, а потом вставлять его в другое место(текста, разумеется). Собственно здесь у нас было выполнено две команды: копирование и вставка. 
Как создать такой макрос? С помощью Команды - очень просто:
```
public class Macros implements Command{
	private List<Command> commands;
	
	public void setCommands(List<Command> commands){
		this.commands = commands;
	}
	
	public void execute(){
		for(Command next: commands){
			next.execute();
		}
	}
}
```
Как вы видите, в данном примере макрос - это тоже Команда. Просто его execute() будет вызывать соответствующие методы у других Команд. Таким образом можно "записать" этот макрос и повторять переданную ему последовательность команд в строго определенном порядке. 
Операция отмены в данном случае будет так же тривиальной:
```
public class Macros implements Command{
	private List<Command> commands;
	
	public void setCommands(List<Command> commands){
		this.commands = commands;
	}
	
	public void execute(){
		for(Command next: commands){
			next.execute();
		}
	}
	
	public void undo(){
		for(int i = commands.size() - 1; i >= 0; i--){
			commands.get(i).undo();
		}
	}
}
```
Все элементарно: мы просто вызываем undo у каждой команды, которая имеется в макросей. В обратном порядке, конечно же. 
Многие из нас, когда приходят домой, включают свет при входе, а позже и телевизор(или компьютер ;)). Эти серые будни у нас повторяются ежедневно, посему можно было бы придумать кнопочку, которая включает нам сразу все, что нам нужно, вместо того, чтоб делать это поотдельности. Тогда наш выключатель по сути не изменится, хотя добавилась теперь возможность выключать по средством всего одной команды:
```
public class Switch {
    private Command onCommand;

    public Switch(Command onCommand) {
         this.onCommand = onCommand;
    }

    public void on() {
         onCommand.execute();
    }

    public void off() {
         offCommand.undo();
    }
}
```
Вот только команды ему будут передаваться непростые. Это наш вышеприведенный макрос, немного видоизмененный, правда:
```
public class Macros implements Command{
	private List<Command> commands;
	
	public Macros(List<Command> commands){
		this.commands = commands;
	}
	
	public void execute(){
		for(Command next: commands){
			next.execute();
		}
	}
	
	public void undo(){
		for(int i = commands.size() - 1; i >= 0; i--){
			commands.get(i).undo();
		}
	}
}
```
А вот как мы теперь это все используем:
```
public class Main{
	public static void main(String []args){
		TV tv = new TV();
		Light light = new Light();
	
		Command tvOn = new TvOnCommand(tv);
		
		Command lightOn = new LightOnCommand(light);
		
		List<Command> onCommands = new ArrayList<Command>();
		commands.add(tvOn);
		commands.add(lightOn);
		
		Macros onMacros = new Macros(onCommands);
		
		Switcher switcher = new Switcher(onMacros);
	}
}
```
Все, теперь наш выключатель работает не просто с каким-то прибором, а с целой серией приборов. При этом он даже об этом не подозревает.

Теперь коротко о применениях Команды..

**Transactional behavior**
Часто бывают такие ситуации, чтоб какой-то процесс или выполнился полностью, или вообще не выполнился. Это так называемые транзакции. Допустим, вы разрабатываете какой-то инсталлятор, он должен устанавливать ваше приложение, однако, если произошла какая-то проблема, он должен откатить свои изменения до 0.
Это решается следующим образом: 
1. Каждый этап установки - это Команда, которая имеет работоспособный undo().
2. Все команды храняться в стеке(можно в макросе).
3. При поломке, делаем undo() для всех команд, как это было в случае вышеприведенного макроса.
Собственно это и все, что я хотел сказать по этой теме :)

**Wizards**
Все мы сталкивались с wizard'ами для установки ПО, для его конфигурации и тому подобное. Команда и здесь сует свой нос! Дело в том, что в этих wizard'ах есть кнопка Назад, которая возвращает на предыдущий этап, причем данные из этого этапа не потерялись - они заполнились на этой формочке наново. Как же это сделать удобно для разработчика? 
А следует это делать так: данный из каждого этапа помещаются в соответствующую Команду, при этом никакого выполнения команды конечно же не происходит. При нажатии кнопки "Ходiмо Узад" данные из команды вытягиваются и суются снова на форму. Когда мы дошли до конца, где есть кнопочка "Ну усьо, приехали", для всех команд выполняется execute(). Подробней смотрим пункт про транзакции. 

**Command & SWING**
Дабы привести пример Команды в жизни, обратимся к SWING(вообще штука, полностью построенная на шаблонах. Респект разработчикам Sun'a). В SWING Команду можно встретить в иерархии классов, начало которым берет Action - это класс, который добавляется, к примеру, в Меню или в Тулбар кнопочки, таким образом она обеспечивает одинаковую команду для нескольких компонентов. При изменении команды, изменятся все подвязанные к ней компоненты. Однако Команда она потому, что у нее есть метод actionPerformed() - аналог наших execute(). Стоит, конечно, заметить, что в данном случае мы наблюдаем переплетение двух шаблонов - Наблюдателя(Observer) и всем знакомой Команды. Вот яркий пример, когда можно запутаться в шаблонах, если расчитывать, что в одном месте может присутствовать всего один шаблон :)

**Command vs. Multi-threading**
Часто, для снижения нагрузки на основной поток, команды раздают другим потокам. Делается это так: наш Invoker имеет ссылку на пул потоков, проходится по нескольким потокам и раздает все Команды, которые у него имеются. В данном случае поток даже не знает(во всяком случае не должен) о том, что он выполняет - он просто обворачивает команду и выполняет ее. Замечу, что вполне работоспособный сценарий, когда не Invoker передает потокам команды, а есть какой-то объект-посредник, который отбирает у Invoker'a объекты команд и передает потокам. В данном случае нужно так же не переборщить, т.к. не забываем, что создание потока - далеко не малопотребительная операция.

**Logging**
В данном случае мы говорим не про обычный вывод сообщений в лог. В данном контексте, логирование - это запись этапов работы приложения с целью его последующего восстановления. Чтоб добиться такого эффекта, нужно разбить выполнение приложения на какие-то команды, которые можно записывать на HDD(хотя бы той же сериализацией). Тогда, если приложение записывало каждый свой шаг таким вот последовательным логированием, а потом тетя Клава пролила на проводку немножко мыльной воды, и сервер упал, мы можем восстановить состояние нашего детища. Как? Да обычно - читаем эти команды с HDD, создаем их объекты и проходимся по execute() каждой команды последовательно. Вот, собственно, и вся магия. Хотя на самом деле все так просто лишь в теории - добиться такой архитектуры в крупных приложениях очень тяжело.

Заключительная часть..

**Command, Action etc.**
Часто возникает путанница с Командой из-за названий. Во-первых, Command и Action - это одно и то же, и не верьте тем злодеям, кто разъединяет эти два понятия. Во-вторых, Action в Struts & commandObject в Spring MVC - это не примеры шаблона Команды, это просто такие названия. Этим злодеям, которые утверждают обратное, тоже верить не следует.
1. Client, Source, Invoker.
2. Command Object, Action Object, Event Object.
3. Receiver, Target.

Это примеры названий, который могут быть примером Команды, а могут и не быть по причине существования синонимов и омонимов(так что все претензии - к филологам, они всему виной). Так что будьте осторожны в терминологии, связанной с Командой. Принятые слоглашения ключевых объектов данного шаблона - это Client, Invoker, Command object, Receiver.

**Правильный Command. Best Practice**
1. Command Object не должен делать все вместо получателя - слишком много обязанностей для него. При нарушении этого принципа нарушается [SRP-принцип](http://en.wikipedia.org/wiki/Single_responsibility_principle), а так же усложняется процесс создания undo() операции. Лучшая практика показывает, что Command Object должен быть настолько прост, насколько это возможно.
2. Invoker не должен знать о Reciever'e. Главное, что предоставляет нам данный шаблон, - это отвязка Invoker'a от Reciever'а. То есть первый вообще не должен знать о последнем. Это изюминка Команды и если нарушается этот принцип, не называйте свою конструкцию Командой.
3. Invoker не должен знать о конкретных классах Команд. Если Invoker знает какую команду он дергает, значит он знает слишком много(таких персонажей обычно убивают в фильмах). То есть по сути Вызыватель будет знать о том, что делает команда, а это неправильно, он должен ведать только о базовом, абстрактном классе.

Ну, собственно, это и весь рассказ.

**Литература**:
* [Head First Design Pattern](http://books.google.com.ua/books?id=LjJcCnNf92kC&dq=head+first+design+patterns)
* [oodesign.com](http://www.oodesign.com/command-pattern.html)
* [Wikipedia](http://en.wikipedia.org/wiki/Command_pattern)
* [JT Webinar](http://javatalks.ru/ftopic12458.php)

А также многие другие тропинки гугла.