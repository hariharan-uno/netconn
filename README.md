# netconn
Example to show how to mock a net.Conn for testing

There are different ways to mock an interface in Go. One of them is to create a mock struct and implement the methodset of the interface. But, for large interfaces, it gives rise to tedious code and unnecessary implementation of all the boring methods. This exxample specifically deals with mocking net.Conn interface, using minimal code necessary. But the concept can be generalized to mock any interface.

We create servers in Go using a code similar to this:

```go
ln, err := net.Listen("tcp", ":8080")
if err != nil {
  // handle error
}
for {
  conn, err := ln.Accept()
  if err != nil {
    // handle error
  }
  go handleConnection(conn)
}
```

To test handleConnection, we can create a mock net.Conn like this:

```go
// mockConn represents a mock net.Conn. It has the method sets of net.Conn.
type mockConn struct {
	// We are embedding this interface because we implement
	// only a subset of the methods of net.Conn on mockConn.
	// Doing this, we don't have implement the unwanted methods
	// for testing.
	net.Conn

	// If we embed *bytes.Buffer directly, the Read method
	// doesn't resolve, as there is also a Read method in net.Conn
	b *bytes.Buffer
}

func (m mockConn) Read(b []byte) (n int, err error) {
	return m.b.Read(b)
}

// Close is a no-op. It is required to implement the method set of net.Conn.
func (m mockConn) Close() error {
	return nil
}

// The following are just for convenience, not necessary to implement.

func newMockConn() mockConn {
	return mockConn{
		b: new(bytes.Buffer),
	}
}

func (m mockConn) WriteString(s string) error {
	_, err := m.b.WriteString(s)
	return err
}
```
