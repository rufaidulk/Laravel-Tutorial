# Laravel Notes - Master Layout

A master layout consists header, footer, sidebar that can be inheritted to all pages.
```html
<html>
<head>
    <title> @yield('title') </title>
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.4\/css/bootstrap.min.css">
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.4\/css/bootstrap-         theme.min.css">
    <script src="//code.jquery.com/jquery-1.11.3.min.js"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.4/js/bootstrap.mi\n.js"></script>
</head>
<body>
@include('shared.navbar')
@yield('content')
</body>
</html>
```

- use **@yield** directive to insert title from another section
- use **@include** directive to embed other views to master layout.
- use **@yield** directive to embed section called **content** from other sections.
## Extending the master layout
```html
@extends('master')
@section('title', 'Home')
@section('content')
    <div class="container">
        <div class="content">
            <div class="title">Home Page</div>
            <div class="quote">Our Home page!</div>
        </div>
    </div>
@endsection
```
