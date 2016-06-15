# Angular 2 Chat

Multi-user, multi-room and real-time chat

Angular 2 + Socket.io + NodeJS + ExpressJS + MongoDB + Webpack + Gulp + SASS

### by Jussi Kinnula

https://github.com/jussikinnula/angular2-socketio-chat-example

https://jussikinnula.github.io/angular2-chat

---

# Server

___

### Server

`src/server.ts`

```
// Instantiate server app
let app = require("./server/app");
```

TypeScript modules are used also on server side, so we only need to require the first part as a "NodeJS module".

___

### Server

`src/server/app.ts`

```
import * as express from "express";
import * as http from "http";
import * as serveStatic from "serve-static";
import * as path from "path";
import * as dotenv from "dotenv";
import * as socketIo from "socket.io";
import * as mongoose from "mongoose";
```

Dependencies for the server app.

___

### Server

`src/server/app.ts`

```
class Server {
    public app: any;
    private server: any;
    private io: any;
    private mongo: any;
    private root: string;
    private port: number;

    ...
}
```

Properties, basically references to instances.

___

### Server

`src/server/app.ts`

```
class Server {
    ...
    public static bootstrap(): Server {
        return new Server();
    }
    ...
}

// Bootstrap the server
let server = Server.bootstrap();
export = server.app;
```

Bootsrapping is exporting a NodeJS module (needed for `src/server.ts` to require the app).

___

### Server

`src/server/app.ts`

```
class Server {
    ...
    constructor() {
        this.app = express();
        this.config();
        this.routes();
        this.server = http.createServer(this.app);
        this.databases();
        this.sockets();
        this.listen();
    }
    ...
}
```

Server app initialises everything in order - first create ExpressJS app, then do configurations, setup routes, create HTTP server, initialize MongoDB, Socket.io and finally start listening on user specified port.

___

### Server

`src/server/room-socket.ts`

```
class RoomSocket {
    ...
    constructor(private io: any) {
        this.nsp = this.io.of("/room");
        this.nsp.on("connection", (socket: any) => {
            console.log("Client connected");
            this.socket = socket;
            this.listen();
        });
    }
    ...
}
```

Socket.io connection is separated to mounts (there's `/room` for getting room list and other interaction with rooms, and `/messages/XXX` for messaging.

___

### Server

`src/server/room-socket.ts`

```
class RoomSocket {
    ...
    private listen(): void {
        this.socket.on("disconnect",
            () => this.disconnect());
        this.socket.on("create",
            (name: string) => this.create(name));
        this.socket.on("remove",
            (name: string) => this.remove(name));
        this.socket.on("list",
            () => this.list());
    }
    ...
}
```

Per-user connection listens `create`, `remove` and `list` signals + handles disconnection.

___

### Server

`src/server/room-socket.ts`

```
class RoomSocket {
    ...
    private create(name: string): void {
        Room.create({
            name: name,
            created: new Date(),
            messages: []
        }, (error: any, room: IRoom) => {
            if (!error && room) {
                this.createRoom(room);
            }
        });
    }
}
```

Room is created with Mongoose models creator, and when the data is in database. If everything went successfully, further setup is done on `createRoom()` -method.

___

### Server

`src/models/room.model.ts`

```
export interface IRoom {
    name: string;
    created: Date;
}

export interface IRoomModel extends IRoom, mongoose.Document {}

export var RoomSchema = new mongoose.Schema({
    name: {
        type: String,
        unique: true
    },
    created: Date
});

export var Room = mongoose.model<IRoomModel>("Room", RoomSchema);
```

___

### Server

`src/server/room-socket.ts`

```
class RoomSocket {
    ...
    private createRoom(room: IRoom): void {
        if (!this.rooms[room.name]) {
            this.rooms[room.name] =
                new MessageSocket(this.io, room.name);
        }
        this.nsp.emit("create", room);        
    }
    ...
}
```

New `MessageSocket` placeholder is created for the room and newly added room is emitted to the clients.

---

![nice cat picture](./images/angular2-chat-cat.jpg)

---

# Client

___

### Client

`src/client.ts`

```
import { enableProdMode, provide } from "@angular/core";
import { bootstrap } from "@angular/platform-browser-dynamic";
import { ROUTER_PROVIDERS } from "@angular/router-deprecated";
import { AppComponent } from "./client/app";
import { SERVICE_PROVIDERS } from "./client/services/providers";
import "./styles/styles.scss";

if (process.env.NODE_ENV === "production") {
    enableProdMode();
}

bootstrap(AppComponent, [
    ROUTER_PROVIDERS,
    SERVICE_PROVIDERS
]).catch((error: Error) => console.error(error));
```

___

### Client

`src/client/app/app.component.ts`

```
import { Component } from "@angular/core";

import {
    ControlComponent,
    NicknameComponent,
    RoomsComponent
} from "../components";

import {
    RoomService,
    UserService
} from "../services";

declare var require;
const styles: string = require("./app.component.scss");
const template: string = require("./app.component.html");
```

___

### Client

`src/client/app/app.component.ts`

```
@Component({
    selector: "app",
    styles: [styles],
    directives: [
        ControlComponent,
        NicknameComponent,
        RoomsComponent
    ],
    providers: [
        RoomService
    ],
    template
})
```

___

### Client

`src/client/app/app.component.ts`

```
export class AppComponent {
    /**
     * Constructor.
     *
     * @class AppComponent
     * @constructor
     * @param userService UserService
     */
    constructor(public userService: UserService) {}
}
```

___

### Client

`src/client/app/app.component.html`

```
<h1>Angular 2 Chat</h1>

<nickname *ngIf="
    !userService.nickname
    || userService.nickname === ''
"></nickname>

<control *ngIf="
    userService.nickname
    && userService.nickname !== ''
"></control>

<rooms *ngIf="
    userService.nickname
    && userService.nickname !== ''
"></rooms>
```

___

### Client

`src/client/components/control/
    control.component.ts`

```
import { Component } from "@angular/core";
import { UserService, RoomService } from "../../services";
import { IRoom } from "../../../models/room.model";

declare var require;
const styles: string = require("./control.component.scss");
const template: string = require("./control.component.html");

@Component({
    selector: "control",
    directives: [],
    styles: [styles],
    template
})
```

___

### Client

`src/client/components/control/
    control.component.ts`

```
export class ControlComponent {
    room: string = "";
    newRoom: string = "";

    constructor(public roomService: RoomService) {}

    join(): void {
        this.roomService.join(this.room);
    }

    create(): void {
        this.roomService.create(this.newRoom);
        this.newRoom = "";
    }
    ...
}
```

`ControlComponent` just uses RoomService to hook events.

___

### Client

`src/client/components/control/
    control.component.ts`

```
export class ControlComponent {
    ...

    remove(): void {
        this.roomService.remove(this.room);
        this.room = "";
    }

    eventHandler(event: KeyboardEvent): void {
        if (event.key === "Enter") {
            this.create();
        }
    }
}
```

___

### Client

`src/client/components/control/
    control.component.html`

```
<span>
    <select [(ngModel)]="room">
        <option value="">Select room...</option>
        <option
            *ngFor="let room of roomService.rooms | async"
            [value]="room.name"
        >
            {{room.name}}
        </option>
    </select>
    <button (click)="join()">Join</button>
    <button (click)="remove()">Remove</button>
</span>
...
```

`ControlComponent` methods `join()` and `remove()` can interact with the item selected in `<select> ... </select>`.
___

### Client

`src/client/components/control/
    control.component.html`

```
...
<span>
    New room:
    <input
        [(ngModel)]="newRoom"
        (keypress)="eventHandler($event)"
    >
    <button (click)="create()">Create</button>
</span>
```

`ControlComponent` method `create()` can be called by clicking a button or by pressing enter when being in text input.

___

### Client

`src/client/services/room/room.service.ts`

```
import { Injectable } from "@angular/core";
import { ReplaySubject } from "rxjs";
import { List } from "immutable";

import { SocketService, ISocketItem } from "../socket";

import { UserService } from "../user";

import { IRoom } from "../../../models/room.model";
```

___

### Client

`src/client/services/room/room.service.ts`

```
@Injectable()
export class RoomService {
    rooms: ReplaySubject<any> = new ReplaySubject(1);
    private list: List<any> = List();

    constructor(
        private socketService: SocketService,
        private userService: UserService
    ) {
        this.socketService
            .get("room")
            .subscribe( ... );
    }
    ...
}
```

`SocketService` services `get()` method returns an Observable.

___

### Client

`src/client/services/room/room.service.ts`

```
(socketItem: ISocketItem) => {
    let room: IRoom = socketItem.item;
    let index: number = this.findIndex(room.name);
    if (socketItem.action === "remove") {
        this.list = this.list.delete(index);
    } else {
        if (index === -1) {
            this.list = this.list.push(room);
        } else {
            this.list = this.list.set(index, room)
        }
    }
    this.rooms.next(this.list);
},
error => console.log(error)
```

The Observable is subscribed, and removal, creation and updating of room items is handled.

___

### Client

`src/client/services/room/room.service.ts`

```
export class RoomService {
    ...
    join(name: string): void {
        for (let roomIndex in this.userService.rooms) {
            let room = this.userService.rooms[roomIndex];
            if (room.name === name) { return; }
        }
        let index = this.findIndex(name);
        if (index !== -1) {
            let room = this.list.get(index);
            this.userService.rooms.push(room);
        }
    }
    ...
}
```

Joining a room just adds the room to `UserService` services `rooms` array (it's just an array of `IRoom` objects).

___

### Client

`src/client/services/room/room.service.ts`

```
export class RoomService {
    ...
    leave(name: string) {
        // First remove the room from user joined rooms
        for (var i = 0; i < this.userService.rooms.length; i++) {
            let room = this.userService.rooms[i];
            if (room.name === name) {
                this.userService.rooms.splice(i, 1);
            }
        }
    }
    ...
}
```

Removing a room just removes the `IRoom` object from `UserService` `rooms` array.

___

### Client

`src/client/services/room/room.service.ts`

```
export class RoomService {
    ...
    private findIndex(name: string): number {
        return this.list.findIndex((room: IRoom) => {
            return room.name === name;
        });
    }
    ...
}
```

Helper function to search for room with `name`.

___

### Client

`src/client/services/socket/socket.service.ts`

```
export class SocketService {
    ...
    get(name: string): Observable<any> {
        this.name = name;
        this.socket = io.connect(this.host + "/" + this.name);
        this.socket.on("connect", () => this.connect());
        this.socket.on("disconnect", () => this.disconnect());
        this.socket.on("error", (error: string) => {
            console.log(`ERROR: "${error}"`);
        });

        return Observable.create((observer: any) => { ... });
    }
    ...
}
```

`SocketService` services `get()` initiates the Socket.io connection + handles `connect()`, `disconnect()` and errors - and creates an Observable.

___

### Client

`src/client/services/socket/socket.service.ts`

```
return Observable.create((observer: any) => {
    this.socket.on("create",
        (item: any) => observer.next({
            action: "create",
            item: item
        }) );
    this.socket.on("remove",
        (item: any) => observer.next({
            action: "remove",
            item: item
        }) );
    return () => this.socket.close();
});
```

Types of action needs to be defined, so those can be handled on `RoomService` and `MessageService`.

---

![nice cat picture 2](./images/angular2-chat-cat-2.jpg)

---

# The code

https://github.com/jussikinnula/angular2-socketio-chat-example

---

# And the deployed version

https://ng2chat.herokuapp.com/

---

# Thank you!