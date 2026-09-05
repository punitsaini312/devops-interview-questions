Docker CMD vs ENTRYPOINT

CMD

CMD provides the default command for a container.

FROM alpine
CMD ["echo", "Hello from CMD"]

Run:

docker run --rm cmd-demo

Output:

Hello from CMD

CMD can be overridden

docker run --rm cmd-demo date

Now Docker runs date instead of the default CMD.

Think:

CMD = "This is my default command, but you can replace it."

What does --rm mean?

docker run --rm cmd-demo

--rm means Docker automatically removes the container after it stops.

Without --rm:

Create → Run → Stop → Container remains

With --rm:

Create → Run → Stop → Container deleted

It is useful for temporary/test containers.

ENTRYPOINT

ENTRYPOINT defines the main program that the container is intended to run.

FROM alpine
ENTRYPOINT ["echo"]

Run:

docker run --rm entrypoint-demo "Hello"

Docker runs:

echo Hello

Unlike CMD, the value after the image name normally does not replace the ENTRYPOINT. It is passed as an argument.

Think:

ENTRYPOINT = "This is the main program."

CMD + ENTRYPOINT Together

This is the most useful pattern to remember:

FROM alpine

ENTRYPOINT ["echo"]
CMD ["Hello"]

Run:

docker run --rm demo

Result:

Hello

Conceptually:

ENTRYPOINT = echo
CMD        = Hello

docker run demo
       ↓
echo Hello

If we run:

docker run --rm demo World

Result:

World

Conceptually:

ENTRYPOINT = echo
CMD        = Hello

docker run demo World
       ↓
echo World

So:

ENTRYPOINT = main program
CMD        = default argument(s)

CMD vs ENTRYPOINT — Interview Answer

Question: What is the difference between CMD and ENTRYPOINT?

"Both are used to define what runs when a container starts. CMD provides a default command or default arguments, and it can easily be overridden when we run the container. ENTRYPOINT is used when we want to define the main program that the container should run. If we provide something after the image name, it is normally passed as an argument to the ENTRYPOINT instead of replacing it."

Easy example

ENTRYPOINT ["python"]
CMD ["app.py"]

Normally:

docker run myapp

runs:

python app.py

But:

docker run myapp test.py

runs:

python test.py

Remember

ENTRYPOINT = main program
CMD        = default command/arguments

CMD can be replaced. ENTRYPOINT normally stays and receives the provided arguments.
