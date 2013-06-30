I just created a very simple timer. It is just a wraper around the gettimeofday() funtion, but it make the code using it so much simpler and nice. I enjoyed writing this, and you are invited to improve this if you have any nice idea. --General/NirSoffer

**Typical Usage:**
    
#import "General/NIRTimer.h"

General/NIRTimer *timer = General/[[NIRTimer alloc] init];

[timer start];
[someObject doLongOperation];
[timer stop];

General/NSLog(@"doLongOperation: %6.3f seconds", [timer seconds]);

[timer start];
[someObject doMore];
[timer stop];

General/NSLog(@"doMore: %6.3f seconds", [timer secondsSinceStart]);
General/NSLog(@"Total: %6.3f seconds", [timer seconds]);




----

Here is the code:
    
//
//  General/NIRTimer.h
//  General/DirectionService
//
//  Created by Nir Soffer on Wed Jun 04 2003.
//  Copyright (c) 2003 Nir Soffer. All rights reserved.
//


#import <Foundation/Foundation.h>


@interface General/NIRTimer : General/NSObject
{
    long started;
    long time;
    BOOL isRunning;
}

// reset
- (void)reset;
- (void)resetWithSeconds:(double)newSeconds;
- (void)resetWithMicroseconds:(long)newMicroseconds;

// starting and stoping
- (void)start;
- (void)stop;

// reporting total time
- (long)microseconds;
- (double)seconds;

// reporting elapsed time - from last start
- (long)microsecondsSinceStart;
- (double)secondsSinceStart;

@end


//
//  General/NIRTimer.m
//  General/DirectionService
//
//  Created by Nir Soffer on Wed Jun 04 2003.
//  Copyright (c) 2003 Nir Soffer. All rights reserved.
//

#import "General/NIRTimer.h"
#import <sys/time.h>

@interface General/NIRTimer (General/NIRTimerPrivate)
- (long)_currentTime;
@end

@implementation General/NIRTimer

// reset
- (void)reset
{
    // Reset with zero microsends
    [self resetWithMicroseconds:0];
}

- (void)resetWithSeconds:(double)newSeconds
{
    // Convert to microseconds and reset with microseconds
    long newMicroseconds = lround(newSeconds * 1000000);
    [self resetWithMicroseconds:newMicroseconds];
}

- (void)resetWithMicroseconds:(long)newMicroseconds
{
    // main reset
    if (isRunning) isRunning = NO;
    time = newMicroseconds;
}

// starting and stoping
- (void)start
{
    if (isRunning) return;
    
    started = [self _currentTime];
    isRunning = YES;
}

- (void)stop
{
    if (! isRunning) return;

    time = time + [self _currentTime] - started;
    isRunning = NO;
}

// reporting time
- (long)microseconds
{
    if (isRunning){
        // report the current total, like -stop, without stoping
        return time + [self _currentTime] - started;       
    } else {
        // report the time recorded
        return time;
    }
}

- (double)seconds
{
    return [self microseconds] / 1000000.0;
}

// reporting elapsed time - from last start
- (long)microsecondsSinceStart
{
    if (isRunning){
        // report the time from the last start
        return [self _currentTime] - started;
    } else {
        // report the time recorded
        return time;
    }
}

- (double)secondsSinceStart
{
    return [self microsecondsSinceStart] / 1000000.0;
}

@end

@implementation General/NIRTimer (General/NIRTimerPrivate)

- (long)_currentTime
{
    struct timeval tp;
    gettimeofday(&tp, NULL);
    return (tp.tv_sec * 1000000 + tp.tv_usec);
}

@end
