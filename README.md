# gear-ratelimiter
Smart rate limiter middleware for Gear.


## Example with gear
    func init() {
        options = &smartlimiter.Options{
            GetID: func(req *http.Request) string {
                ra, _, _ := net.SplitHostPort(req.RemoteAddr)
                return net.ParseIP(ra).String()
            },
            Max:      10,
            Duration: time.Minute, // limit to 1000 requests in 1 minute.
            Policy: map[string][]int{
                "GET /a": []int{3, 5 * 1000},
                "GET /b": []int{5, 60 * 1000},
                "/c":     []int{6, 60 * 1000},
            },
            RedisAddr: "127.0.0.1:6379",
        }
    }
    func main() {
        app := gear.New()
        logger := &middleware.DefaultLogger{W: os.Stdout}
        app.Use(middleware.NewLogger(logger))

        //Add rate limiter middleware
	    app.Use(smartlimiter.NewLimiter(options))

        router := gear.NewRouter()
        router.Get("/", func(ctx *gear.Context) error {
            return ctx.HTML(200, "<h1>Hello, Gear!</h1>")
        })
        router.Get("/a", func(ctx *gear.Context) error {
            return ctx.HTML(200, "<h1>Hello, Gear! /a</h1>")
        })
        router.Get("/b", func(ctx *gear.Context) error {
            return ctx.HTML(200, "<h1>Hello, Gear! /b</h1>")
        })
        router.Get("/c", func(ctx *gear.Context) error {
            return ctx.HTML(200, "<h1>Hello, Gear! /c</h1>")
        })
        app.UseHandler(router)
        app.Error(app.Listen(":3000"))
    }
