## LLD Design Patterns

# Creational Patterns (Object Creation)
Singleton Pattern #################################################################

Intuition: Ensure a class has only one instance and provide a global access point to it.
Code Intuition: Make constructor private so new instance can't be created anywhere else and then expose a static method which actually creates and returns the instance. So anywhere this static method is getting called, same instance will be returned.

Examples usages: 
database connection pool
configuration manager
logger
cache manager
application state manager

* Step 1 — Private constructor

Prevent direct object creation.

class Logger {

  private static instance: Logger

  private constructor() {}
}

* Step 2 — Static instance getter
class Logger {

  private static instance: Logger

  private constructor() {}

  static getInstance(): Logger {

    if (!Logger.instance) {
      Logger.instance = new Logger()
    }

    return Logger.instance
  }

  log(message: string) {
    console.log(message)
  }
}

Step 3 — Using Singleton
const logger1 = Logger.getInstance()
const logger2 = Logger.getInstance()

logger1.log("hello")
logger2.log("world")
console.log(logger1 === logger2)         //True


Factory Method Pattern #################################################################

Isolates the Object cretion. Pass type and get the object you want to create. 

Abstract Factory Pattern #################################################################

Intuition: Factory creates a FAMILY of related objects

Abstract Factory Interface

interface UIFactory {
  createButton(): Button
  createCheckbox(): Checkbox
}
Concrete Factory

Windows factory:

class WindowsFactory implements UIFactory {
  createButton(): Button {
    return new WindowsButton()
  }

  createCheckbox(): Checkbox {
    return new WindowsCheckbox()
  }
}

Mac factory:

class MacFactory implements UIFactory {
  createButton(): Button {
    return new MacButton()
  }

  createCheckbox(): Checkbox {
    return new MacCheckbox()
  }
}

Usage:

const factory = new WindowsFactory()

const button = factory.createButton()
const checkbox = factory.createCheckbox()

 Builder Pattern #################################################################

Controls Constructor Explosion when creating objects, dont have to pass so many values while creating objects.
Intuition: Create methods which returns the same type by attaching the field.
Main components: Object (Ride), Builder (RideBuilder), Method(Build)
Example:  
Ride ride =
    new RideBuilder()
        .rideId("R123")
        .user(user)
        .pickup(pickup)
        .drop(drop)
        .vehicleType(SUV)
        .build();
Use cases: complete object being built in multiple phases | constructor has many params  

# Structural Patterns (Class relationships)

 Adapter Pattern #################################################################

When to Use Adapter:
You should think Adapter Pattern when:
You want to use an existing class but its interface does not match your system
Example situations:
* external API integration
* legacy code integration
* third-party libraries
* SDKs

Code Intuition: Write a wrapper on top of external components like: 
For Payments
* Razorpay -> razorpay.pay()
* Stripe -> stripe.makePayment()
* Paytm -> paytm.sendMoney()
To integrate these all will have to write so many if conditions/tight coupling without Adapter pattern.
With Adapter pattern: Write an Interface like
interface Paymentgateway {
    pay(amount: number) : void
}
Define Adapters implementing this interface

class StripeAdapter implements Paymentgateway {
  constructor(private stripe: Stripe) {}
  pay(amount: number): void {
    this.stripe.makePayment(amount)
  }
}

class RazorpayAdapter implements Paymentgateway {
  constructor(private razorpay: Razorpay) {}
  pay(amount: number): void {
    this.razorpay.pay(amount)
  }
}

 Decorator Pattern #################################################################

Base object
   ↓
Wrapped by decorators
   ↓
Each decorator adds behavior
Example:
    <!-- const rideDistance = 10;

    let fare: FareComponent = new BaseFare(rideDistance);

    // apply surge pricing
    fare = new SurgePricingDecorator(fare, 1.5);

    // apply night charge
    fare = new NightChargeDecorator(fare, 50);

    console.log("Final Fare:", fare.calculateFare()); 
    -->


 Facade Pattern #################################################################

Intuition: Client shouldn't know too much like In Ride service example. Client
shouldn't call:
fareService.calculateFare(ride)
paymentService.processPayment(ride)
notificationService.sendReceipt(ride)
analyticsService.recordRide(ride)
driverService.markDriverAvailable(driver)

Instead all these should be put behind a single interface (Facade) and client only deals with this interface.
Facade Example:
class RideCompletionFacade {

  constructor(
    private fareService: FareService,
    private paymentService: PaymentService,
    private notificationService: NotificationService,
    private analyticsService: AnalyticsService,
    private driverService: DriverService
  ) {}
  completeRide(distance: number) {
    const fare = this.fareService.calculateFare(distance)
    this.paymentService.processPayment(fare)
    this.notificationService.sendReceipt()
    this.analyticsService.recordRide()
    this.driverService.markDriverAvailable()
    console.log("Ride completed successfully")
  }
}

* Client Usage
Client code becomes extremely simple.
const facade = new RideCompletionFacade(
  new FareService(),
  new PaymentService(),
  new NotificationService(),
  new AnalyticsService(),
  new DriverService()
)
facade.completeRide(10)

# Behavioral Patterns (Communication)

 Strategy Pattern ################################################################# 

When behaviour changes per type like - UPIPayment, CardPayment, CashPayment. Have a BaseStrategy then extend to various types.

 Observer Pattern #################################################################

One object changes state and multiple other objects need to react automatically (Solves tight coupling of Classes like Ride and NotificationService)
Main components: 
1. Subject (Publisher), 
2. Observer (Subscriber), 
3. Subscription mechanism
    Observers register with the subject.
    subscribe()
    unsubscribe()
    notify()

Example: 
RideService.completeRide()
        ↓
RideEventPublisher.notifyRideCompleted()
        ↓
NotificationService.onRideCompleted()
BillingService.onRideCompleted()
AnalyticsService.onRideCompleted()

 Command Pattern #################################################################

Use this when you want to do some actions as commands.

Mental model:
CommandInvoker
   ↓
ProcessPaymentCommand
   ↓
PaymentService

Code Intuition:

Command Interface
interface Command {
  execute(): Promise<void>
}

These services perform the real work:

class EmailService {
  async sendEmail(to: string, message: string) {
    console.log(`Sending email to ${to}: ${message}`)
  }
}
class ImageService {
  async resizeImage(file: string) {
    console.log(`Resizing image: ${file}`)
  }
}

Concrete Command Implementations:

Email Command
class SendEmailCommand implements Command {

  constructor(
    private emailService: EmailService,
    private to: string,
    private message: string
  ) {}

  async execute(): Promise<void> {
    await this.emailService.sendEmail(this.to, this.message)
  }
}

Image Processing Command
class ResizeImageCommand implements Command {

  constructor(
    private imageService: ImageService,
    private file: string
  ) {}

  async execute(): Promise<void> {
    await this.imageService.resizeImage(this.file)
  }
}


class JobQueue {

  private queue: Command[] = []

  add(command: Command) {
    this.queue.push(command)
  }

  async process() {
    while (this.queue.length > 0) {
      const command = this.queue.shift()!
      await command.execute()
    }
  }
}


Main: 
const emailService = new EmailService()
const imageService = new ImageService()

const emailCommand = new SendEmailCommand(
  emailService,
  "user@example.com",
  "Your upload is complete"
)

const resizeCommand = new ResizeImageCommand(
  imageService,
  "photo.jpg"
)

const queue = new JobQueue()

queue.add(emailCommand)
queue.add(resizeCommand)

queue.process()


 Chain of Responsibility #################################################################

interface Handler {
  setNext(handler: Handler): Handler
  handle(request: any): void
}

abstract class BaseHandler implements Handler {

  private nextHandler?: Handler

  setNext(handler: Handler): Handler {
    this.nextHandler = handler
    return handler
  }

  handle(request: any): void {
    if (this.nextHandler) {
      this.nextHandler.handle(request)
    }
  }
}

Concrete Handlers:

Logging Handler
class LoggingHandler extends BaseHandler {

  handle(request: any): void {
    console.log("Logging request")

    super.handle(request)
  }
}

Authentication Handler
class AuthHandler extends BaseHandler {

  handle(request: any): void {

    if (!request.user) {
      console.log("Authentication failed")
      return
    }

    console.log("Authentication successful")

    super.handle(request)
  }
}

Rate Limit Handler
class RateLimitHandler extends BaseHandler {

  handle(request: any): void {

    if (request.requests > 100) {
      console.log("Rate limit exceeded")
      return
    }

    console.log("Rate limit passed")

    super.handle(request)
  }
}


Step 4 — Create the Chain
const logger = new LoggingHandler()
const auth = new AuthHandler()
const rateLimiter = new RateLimitHandler()

logger.setNext(auth).setNext(rateLimiter)


Main:
const request = {
  user: "piyush",
  requests: 10
}
logger.handle(request)
