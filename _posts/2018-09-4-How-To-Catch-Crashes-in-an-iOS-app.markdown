---
layout: post
title:  "How-To-Catch-Crashes-in-an-iOS-app"
date:   2018-09-04 18:26:00 -0700
categories: jekyll update
---

Like any software, apps comes with crashes. There are off-the-shelf tools in the market like Crashlytics to catch those crashes and generates nice reports for mobile developers. But how does it do the job? How to record something when an app is crashing? Crashlytics itself is a proprietary software but luckly we have Microsoft which open sourced its app center iOS SDK [Ref: Apple Docs]https://github.com/Microsoft/AppCenter-SDK-Apple. We can see from there that app center actually uses open source project PLCCrashReport to catch and record crashes behind the scene. So I digged into its source code to get answers of my questions.

To understand how to catch the crash, we first need to know what happens to our program when a crash happens. So basically in the Unix like system (POSIX-compliant), IPC signals will be sent to the process of our app to notify the crash occurs. There are a bunch of signals defined in POSIX but we are in particular interest in the following signals:

{% highlight c %}
static int monitored_signals[] = {
    SIGABRT, //(abort)
    SIGBUS,  //(bus error (memory or address error))
    SIGFPE,  //(floating point error (like divide by zero))
    SIGILL,  //(kill)
    SIGSEGV, //(segmentation fault)
    SIGTRAP  //(debugger requested, e.g., when the program hits a break point)
};
{% endhighlight %}

These signals indicates different reasons for the crash (or stop). So what PLCCrashReport does is to register handlers for these signals which will be triggered when the program receives such signals. 


Here is how to register the handler to a signal (signo). It uses the interface defined in POSIX on signals [Ref : link] http://pubs.opengroup.org/onlinepubs/7908799/xsh/sigaction.html
{% highlight c %}
    * @param signo The signal number for which a handler should be registered.
            
    struct sigaction sa;
    struct sigaction sa_prev;
            
    /* Configure action */
    memset(&sa, 0, sizeof(sa));
    sa.sa_flags = SA_SIGINFO|SA_ONSTACK;
    sigemptyset(&sa.sa_mask);
    sa.sa_sigaction = &plcrash_signal_handler;
            
    /* Set new sigaction */
    if (sigaction(signo, &sa, &sa_prev) != 0) {
        int err = errno;
        plcrash_populate_posix_error(outError, err, @"Failed to register signal handler");
        return NO;
    }
    {% endhighlight %}


Here is the handler:

{% highlight c %}
static bool signal_handler_callback (int signal, siginfo_t *info, pl_ucontext_t *uap, void *context, PLCrashSignalHandlerCallback *next) {
    plcrashreporter_handler_ctx_t *sigctx = context;
    plcrash_async_thread_state_t thread_state;
    plcrash_log_signal_info_t signal_info;
    plcrash_log_bsd_signal_info_t bsd_signal_info;
    
    /* Remove all signal handlers -- if the crash reporting code fails, the default terminate
     * action will occur.
     *
     * NOTE: SA_RESETHAND breaks SA_SIGINFO on ARM, so we reset the handlers manually.
     * http://openradar.appspot.com/11839803
     *
     * TODO: When forwarding signals (eg, to Mono's runtime), resetting the signal handlers
     * could result in incorrect runtime behavior; we should revisit resetting the
     * signal handlers once we address double-fault handling.
     */
    for (int i = 0; i < monitored_signals_count; i++) {
        struct sigaction sa;
        
        memset(&sa, 0, sizeof(sa));
        sa.sa_handler = SIG_DFL;
        sigemptyset(&sa.sa_mask);
        
        sigaction(monitored_signals[i], &sa, NULL);
    }

    /* Extract the thread state */
    // XXX_ARM64 rdar://14970271 -- In the Xcode 5 GM SDK, _STRUCT_MCONTEXT is not correctly
    // defined as _STRUCT_MCONTEXT64 when building for arm64; this requires the pl_mcontext_t
    // cast.
    plcrash_async_thread_state_mcontext_init(&thread_state, (pl_mcontext_t *) uap->uc_mcontext);
    
    /* Set up the BSD signal info */
    bsd_signal_info.signo = info->si_signo;
    bsd_signal_info.code = info->si_code;
    bsd_signal_info.address = info->si_addr;
    
    signal_info.bsd_info = &bsd_signal_info;
    signal_info.mach_info = NULL;

    /* Write the report */
    if (plcrash_write_report(sigctx, pl_mach_thread_self(), &thread_state, &signal_info) != PLCRASH_ESUCCESS)
        return false;

    /* Call any post-crash callback */
    if (crashCallbacks.handleSignal != NULL)
        crashCallbacks.handleSignal(info, uap, crashCallbacks.context);
    
    return false;
}
{% endhighlight %}