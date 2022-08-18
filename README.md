# Dependency Injection

## Learning Goals

- Explain dependency injection.
- Use dependency injection in Angular.

## Dependency Injection in Angular

Dependency Injection is a mechanism that lets Angular instantiate the target
objects for us, instead of us doing it manually like we did with the logging
service above.

We still need to import the service in the component class, so that that
component knows about the service class, but now we will remove the line where
we manually create an instance of the service, and instead just define the
`loggingSvce` variable through the component's constructor:

```typescript
import { Component, OnInit } from "@angular/core";
import { LoggingService } from "src/app/logging.service";

@Component({
  selector: "app-send-message-component",
  templateUrl: "./send-message-component.component.html",
  styleUrls: ["./send-message-component.component.css"],
})
export class SendMessageComponentComponent implements OnInit {
  messageString: string;

  constructor(private loggingSvce: LoggingService) {} // declare the loggingSvce variable through the component's constructor

  ngOnInit(): void {}

  onSendMessage() {
    this.loggingSvce.log("Send following message: ");
    this.loggingSvce.log(this.messageString);
  }
}
```

Then we need to add the `LoggingService` class as a "provider" for the
component. Doing so tells Angular that whenever someone asks for a variable of
type `LoggingService`, it should look to provide that instance for the
requester.

While we could add the provider to the component itself, it's better to add it
to the parent module, so that all the child components can use the service. So
our updated `app.module.ts` looks like this:

```typescript
import { NgModule } from "@angular/core";
import { BrowserModule } from "@angular/platform-browser";
import { FormsModule } from "@angular/forms";

import { AppComponent } from "./app.component";
import { HeaderComponentComponent } from "./header-component/header-component.component";
import { ApplicationComponentComponent } from "./application-component/application-component.component";
import { ConversationControlComponentComponent } from "./application-component/conversation-control-component/conversation-control-component.component";
import { ContactListComponentComponent } from "./application-component/contact-list-component/contact-list-component.component";
import { ConversationHistoryComponentComponent } from "./application-component/conversation-history-component/conversation-history-component.component";
import { ConversationThreadComponentComponent } from "./application-component/conversation-history-component/conversation-thread-component/conversation-thread-component.component";
import { SendMessageComponentComponent } from "./application-component/conversation-history-component/send-message-component/send-message-component.component";
import { ContactComponentComponent } from "./application-component/contact-list-component/contact-component/contact-component.component";
import { SenderMessageComponentComponent } from "./application-component/conversation-history-component/conversation-thread-component/sender-message-component/sender-message-component.component";
import { UserMessageComponentComponent } from "./application-component/conversation-history-component/conversation-thread-component/user-message-component/user-message-component.component";

import { LoggingService } from "./logging.service"; // import the new service

@NgModule({
  declarations: [
    AppComponent,
    HeaderComponentComponent,
    ApplicationComponentComponent,
    ConversationControlComponentComponent,
    ContactListComponentComponent,
    ConversationHistoryComponentComponent,
    ConversationThreadComponentComponent,
    SendMessageComponentComponent,
    ContactComponentComponent,
    SenderMessageComponentComponent,
    UserMessageComponentComponent,
  ],
  imports: [BrowserModule, FormsModule],
  providers: [
    LoggingService, // add the service as a provider
  ],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

With these changes, our code works as before, but without our component needing
to create its own instance of the service.

What are the advantage of this approach? There are several:

- The client classes no longer create their own instances, which means we can
  swap out implementations without the client classes needing to be modified. If
  we were explicitly calling the constructor of the instance ourselves, we would
  have to change that code whenever the constructor changed, for example.
- We can inject different versions of the services based on the circumstances -
  this is very useful for mocking for unit testing for example. All we have to
  do is tell Angular about a different context, and then it can build different
  implementations of the services for us.
- Angular’s injector is also a hierarchical injector, which means that it can
  manage the instances of your component’s or module’s services in such a way
  that they are shared between modules and components:
  - Services are always shared with child components, not parent components
    - A module can share its instance of a service with its components
    - A component can share its instance of a service with its child components
    - A component cannot share its instance of a service with its parent
      component
  - You can control this behavior by setting up your service in exactly the same
    way as before, except not defining it in your component’s `providers` array.
    When a service is passed into a component’s constructor, it will be
    initialized to a parent component’s service instance if that component does
    not have its own provider.

In addition to injecting services into components, you can also inject services
into other services. The only extra step is that any service that wants to
benefit from Angular's Dependency Injection functionality must be decorated with
the `@Injectable` annotation. For example, if we had a data service that wanted
to use our logging service, we could declare it in the following way:

```typescript
import { Injectable } from "@angular/core";
import { LoggingService } from "./logging.service";

@Injectable()
export class MessagingDataService {
  constructor(private loggingSvce: LoggingService) {
    loggingSvce.log("Messaging Data Service constructor completed");
  }
}
```

In our examples so far, we have injected the services in the root module, which
meant we could use those services in any of the components in our application.
This is because Angular's Injector service is hierarchical. Whatever level a
service is injected at, it will be available to that component, as well as all
the child components of that component.

The lifecycle of the injected service also changes based on where it is
injected. When a component has a service injected directly at its level, i.e. it
is defined in its own `provider` section, then that component will get its own
instance of that service. This means that when your intent is to use a service
to share data between components, you need to make sure that the service is
injected at a level that is higher than both of the components between which you
are trying to share data.
