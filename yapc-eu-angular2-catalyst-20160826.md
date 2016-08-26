## Angular 2 + Catalyst = <3

#### YAPC::EU 2016

### Jussi Kinnula

@jussikinnula on Twitter

spot@cpan.org

---

# Structure

___

```
angular2-catalyst-starter
├── Changes
├── Makefile.PL
├── Procfile
├── README.md
├── app.psgi
├── cpanfile
├── database
│   └── migrate.pl
├── inc
│   └── Module
│       └── Install.pm
├── lib
│   ├── WebApp
│   │   ├── Controller
│   │   │   ├── Root.pm
│   │   │   └── Todo
│   │   │       └── Base.pm
│   │   ├── Model
│   │   │   └── DB.pm
│   │   ├── Schema
│   │   │   └── Result
│   │   │       └── Todo.pm
│   │   ├── Schema.pm
│   │   └── View
│   │       └── Xslate.pm
│   └── WebApp.pm
├── src
│   ├── app
│   │   ├── app.component.pug
│   │   ├── app.component.spec.ts
│   │   ├── app.component.styl
│   │   ├── app.component.ts
│   │   ├── app.module.ts
│   │   ├── app.routing.ts
│   │   └── index.ts
│   ├── components
│   │   ├── footer
│   │   │   ├── footer.component.pug
│   │   │   ├── footer.component.styl
│   │   │   ├── footer.component.ts
│   │   │   └── index.ts
│   │   ├── header
│   │   │   ├── header.component.pug
│   │   │   ├── header.component.styl
│   │   │   ├── header.component.ts
│   │   │   └── index.ts
│   │   ├── index.ts
│   │   ├── todo-edit
│   │   │   ├── index.ts
│   │   │   ├── todo-edit.component.pug
│   │   │   ├── todo-edit.component.styl
│   │   │   └── todo-edit.component.ts
│   │   └── todo-list
│   │       ├── index.ts
│   │       ├── todo-list.component.pug
│   │       ├── todo-list.component.styl
│   │       └── todo-list.component.ts
│   ├── favicon.ico
│   ├── images
│   │   └── logo.png
│   ├── index.pug
│   ├── main.ts
│   ├── models
│   │   ├── index.ts
│   │   └── todo.model.ts
│   ├── pipes
│   │   ├── date.pipe.ts
│   │   ├── index.ts
│   │   ├── order-by.pipe.ts
│   │   └── pipes.module.ts
│   ├── polyfills.ts
│   ├── services
│   │   ├── index.ts
│   │   ├── services.module.ts
│   │   ├── todo.service.ts
│   │   └── user.service.ts
│   ├── styles
│   │   ├── globals
│   │   │   ├── mixins.styl
│   │   │   ├── normalize.styl
│   │   │   └── variables.styl
│   │   └── main.styl
│   ├── vendor.ts
│   └── views
│       ├── frontpage
│       │   ├── frontpage.routing.ts
│       │   ├── frontpage.view.pug
│       │   ├── frontpage.view.styl
│       │   ├── frontpage.view.ts
│       │   └── index.ts
│       ├── index.ts
│       └── todo
│           ├── index.ts
│           ├── todo.routing.ts
│           ├── todo.view.pug
│           ├── todo.view.styl
│           └── todo.view.ts
├── t
│   ├── 01app.t
│   ├── 02pod.t
│   └── 03podcoverage.t
├── tsconfig.json
├── tslint.json
├── typings.json
├── webapp.pl
└── webpack.config.js
```

---

# Backend

___

`app.psgi`

```perl
use strict;
use warnings;
use utf8;
use lib './lib';
use Env::Heroku::Pg;
use WebApp;
use Plack::Builder;
use Plack::Response;
use Plack::App::File;
use Text::Xslate;
use Encode qw(encode_utf8);
use Data::Dumper;

my $app = WebApp->apply_default_middlewares(WebApp->psgi_app);

my $spa = sub { my ($root, $base) = @_; builder {
  enable 'Head'; enable 'ConditionalGET'; enable 'HTTPExceptions';
  enable 'Static', path => sub{1}, root => $root, pass_through => 1;
  enable_if { $_[0]->{PATH_INFO} } sub {
    my $app = shift; sub {
      my $env = shift;
      my $tx = Text::Xslate->new( path => $root );
      my $res = Plack::Response->new( 200, [
        content_type => 'text/html;charset=utf-8']
      );
      $res->body( encode_utf8( $tx->render(
        'index.html',
        { base => $base }
      ) ) );
      return $res->finalize;
    }
    };
  sub { [ 301, [ Location => $base ], [] ] };
}};

builder {
  enable 'ReverseProxy';

  mount '/assets' => Plack::App::File->new(
    root => "root/assets")->to_app;
  mount '/favicon.ico' => Plack::App::File->new(
    file => "root/favicon.ico")->to_app;
  mount '/rest' => $app;
  mount '/' => $spa->('root','/');
}
```

___

`lib/WebApp/Schema/Result/Todo.pm`

```perl
package WebApp::Controller::Todo::Base;
use Moose;
use namespace::autoclean;
use utf8;
use Scalar::Util qw(looks_like_number);
use DateTime;
use Data::Dumper;

BEGIN { extends 'WebApp::Controller::Root' }

sub todo_base : Chained("rest_base") PathPart("todo") CaptureArgs(0) {
  my ($self, $c) = @_;
  $c->stash->{todos} = $c->model('DB::Todo');
}

sub index : Chained("todo_base") PathPart("") Args(0) ActionClass("REST") {
}

sub index_GET {
  my ($self, $c) = @_;

  my $todos = $c->stash->{todos};

  my @data = ();
  for my $todo ($todos->all) {
    push @data, $todo->formatted_output;
  }

  $self->status_ok( $c, entity => {
    todos => \@data
  });
}

sub index_POST {
  my ($self, $c) = @_;
  my $params ||= $c->req->data || $c->req->params;
  if (
    $params
    and $params->{title}
    and my $todos = $c->stash->{todos}
  ) {
    $params->{created} = DateTime->now();
    my $todo = $todos->create($params);
    $self->status_ok( $c, entity => $todo->formatted_output );
  } else {
    $self->status_not_found($c, message => "invalid parameters");
  }
}

sub stash_todo : Chained("todo_base") PathPart("") CaptureArgs(1) {
  my ($self, $c, $id) = @_;
  if (my $todos = $c->stash->{todos}) {
    $c->stash->{todo} = $todos->find($id);
  }
}

sub todo : Chained("stash_todo") PathPart("") Args(0) ActionClass("REST") {
}

sub todo_GET {
  my ($self, $c) = @_;
  if (my $todo = $c->stash->{todo}) {
    $self->status_ok( $c, entity => $todo->formatted_output );
  } else {
    $self->status_not_found($c, message => "todo not found");
  }
}

sub todo_PUT {
  my ($self, $c) = @_;
  my $params ||= $c->req->data || $c->req->params;
  if (my $todo = $c->stash->{todo}) {
    delete $params->{created};
    $params->{updated} = DateTime->now();
    $todo->update($params) if $params;        
    $self->status_ok( $c, entity => $todo->formatted_output );
  } else {
    $self->status_not_found($c, message => "todo not found");
  }
}

sub todo_DELETE  {
  my ($self, $c) = @_;
  if (my $todo = $c->stash->{todo}) {
    $todo->delete;
    $self->status_ok( $c, entity => { message => 'ok' } );
  } else {
    $self->status_not_found($c, message => "todo not found");
  }
}

__PACKAGE__->meta->make_immutable;

1;
```

___

`lib/WebApp/Schema/Result/Todo.pm`

```
package WebApp::Schema::Result::Todo;
use utf8;
use strict;
use warnings;
use DateTime;
use DateTime::Format::Pg;
use JSON;
use Data::Dumper;
use base 'DBIx::Class::Core';

__PACKAGE__->table("todo");

__PACKAGE__->load_components("InflateColumn::DateTime");

__PACKAGE__->add_columns(
  "id",
  { data_type => "integer", is_auto_increment => 1 },
  "created",
  { data_type => "timestamp with time zone" },
  "updated",
  { data_type => "timestamp with time zone", is_nullable => 1 },
  "title",
  { data_type => "varchar" },
);

__PACKAGE__->set_primary_key("id");

sub formatted_output {
  my ($self) = @_;
  return {
    id => $self->id+0,
    created => $self->created."",
    updated => $self->updated ? $self->updated."" : undef,
    title => $self->title            
  };
}

1;
```

---

# Frontend

___

`src/main.ts`

```
import { enableProdMode } from "@angular/core";
import { platformBrowserDynamic } from "@angular/platform-browser-dynamic";

// App module
import { AppModule } from "./app";

// Global styles
import "./styles/main.styl";

// Declare process variable (so that type checker doesn't nag about it)
declare var process;

// Production mode
if (process.env.NODE_ENV === "production") {
  enableProdMode();
}
```

___

`src/models/todo.model.ts`

```
export interface ITodo {
  id?: number;
  created: Date;
  updated?: Date;
  title: string;
}
```

___

`src/services/todo.service.ts`

```
import { Injectable } from "@angular/core";
import { Http, Response, Headers, RequestOptions } from "@angular/http";
import { Observable, ReplaySubject } from "rxjs";

import { ITodo } from "../models";

@Injectable()
export class TodoService {
  todos: ReplaySubject<any> = new ReplaySubject(1);
  private list: ITodo[];
  private url: string = "/rest/todo";

  constructor(private http: Http) {
    this.listTodos();
  }

  listTodos(): void {
    this.http.get(this.url)
      .map( (response: Response) => response.json() )
      .subscribe(
        (response: any) => {
          this.list = response.todos;
          this.todos.next(this.list);
        },
        (error: any) => console.log(error)
      );
  }

  createTodo(todo: ITodo): void {
    let headers = new Headers({ "Content-Type": "application/json" });
    let options = new RequestOptions({ headers: headers });
    this.http.post(this.url, todo, options)
      .map( (response: Response) => response.json() )
      .subscribe(
        (todo: ITodo) => this.updateOrCreateTodo(todo),
        (error: any) => console.log(error)

      );
  }

  findTodo(id: number): Observable<any> {
    return Observable.create( observer => {
      this.todos.subscribe( todos => {
        const index = this.findIndex(id);
        observer.next(todos[index]);
      });
    });
  }

  updateTodo(todo: ITodo): void {
    let headers = new Headers({ "Content-Type": "application/json" });
    let options = new RequestOptions({ headers: headers });
    this.http.put(this.url + "/" + todo.id, todo, options)
      .map( (response: Response) => response.json() )
      .subscribe(
        (todo: ITodo) => this.updateOrCreateTodo(todo),
        (error: any) => console.log(error)
      );
  }


  removeTodo(todo: ITodo): void {
    this.http.delete(this.url + "/" + todo.id)
      .map( (response: Response) => response.json() )
      .subscribe(
        (success: any) => {
          const index = this.findIndex(todo.id);
          if (index !== -1) {
            this.list.splice(index, 1);
            this.todos.next(this.list);
          }
        },
        (error: any) => console.log(error)

      );
  }

  private updateOrCreateTodo(todo: ITodo) {
    const index: number = this.findIndex(todo.id);
    if (index === -1) {
      // Create
      this.list.push(todo);
    } else {
      // Update
      this.list[index] = todo
    }
    this.todos.next(this.list);        
  }

  private findIndex(id: number): number {
    return this.list.findIndex((todo: ITodo) => {
      return todo.id == id;
    });
  }
}
```

___

`src/components/todo-list/todo-list.component.ts`

```
import { Component } from "@angular/core";
import { Router } from "@angular/router";

import { TodoService } from "../../services";

import { ITodo } from "../../models";

declare var require;
const styles: string = require("./todo-list.component.styl");
const template: string = require("./todo-list.component.pug");

@Component({
  selector: "todo-list",
  styles: [styles],
  template
})

export class TodoListComponent {
  todos: ITodo[];

  constructor(
    public router: Router,
    private todoService: TodoService
  ) {
    this.todoService.todos.subscribe(
      todos => this.todos = todos
    );
  }
}
```

___

`src/components/todo-edit/todo-edit.component.ts`

```
import { Component, OnInit } from "@angular/core";
import { ActivatedRoute, Router } from "@angular/router";

import { TodoService } from "../../services";

import { ITodo } from "../../models";

declare var require;
const styles: string = require("./todo-edit.component.styl");
const template: string = require("./todo-edit.component.pug");

@Component({
  selector: "todo-edit",
  styles: [styles],
  template
})

export class TodoEditComponent implements OnInit {
  selected: ITodo;

  constructor(
    public route: ActivatedRoute,
    public router: Router,
    private todoService: TodoService
  ) {}

  ngOnInit() {
    this.route.params.subscribe( (params: any) => {
      const id = parseInt(params["todo"], 10);
      this.todoService.findTodo(id).subscribe(
        todo => this.selected = todo
      );
    });
  }

  save(title: string) {
    let todo = Object.create(this.selected);
    todo.title = title;
    this.todoService.updateTodo(todo);
    this.router.navigate(["/todo"]);
  }

  remove() {
    this.todoService.removeTodo(this.selected);
    this.router.navigate(["/todo"]);
  }

  cancel() {
    this.router.navigate(["/todo"]);
  }

  create(title: string) {
    let todo: ITodo = {
      created: new Date,
      title: title
    };
    this.todoService.createTodo(todo);
  }
}
```

---

# Running the app

___

You got the instructions and code at:

https://github.com/jussikinnula/angular2-catalyst-starter

---

# Thanks!

You can find the slides at:

https://jussikinnula.github.io/yapc-eu-angular2-catalyst-20160826

@jussikinnula (Twitter)

https://github.com/jussikinnula
