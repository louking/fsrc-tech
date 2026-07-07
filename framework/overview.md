# Shared framework

All of the FSRC apps (scoretility, routetility, contractility, membertility) are based on a python/javascript framework Lou has built up on top of:

- https://datatables.net/
- https://editor.datatables.net/
- https://www.sqlalchemy.org/
- https://flask.palletsprojects.com/en/latest/
- https://jinja.palletsprojects.com/en/latest/
- https://d3js.org/

Server-side processing is done in python via [Flask](https://flask.palletsprojects.com/en/stable/), a microframework, which uses the [Jinja2](https://jinja.palletsprojects.com/en/stable/) template language to render HTML for the browser. Some apps have more server-side processing than others.

## loutilities

https://github.com/louking/loutilities has a lot of "stuff" in it, but the two modules used most of the time are `tables.py` (DataTables/Editor grid integration) and `user/` (shared accounts/roles/SSO). See [loutilities.md](loutilities.md) for the deepened writeup.
