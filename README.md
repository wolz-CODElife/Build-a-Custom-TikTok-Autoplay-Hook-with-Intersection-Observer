# Build a Custom TikTok Autoplay Hook with Intersection Observer
Did you ever imagine how social media like TikTok, Instagram or Twitter detect a particular video post that is in the viewport, autoplay it and stops it immediately it goes out of viewport?
In this article, I will be explaining Intersection Observer which can be used to implement the autoplay/pause feature, create a React custom hook and use it in a TikTok clone.

What is Intersection Observer?

Intersection Observer is JavaScript browser API that asynchronously monitors the position of DOM element with respect to the client’s viewport or a parent/root element. 

How Intersection Observer Works.

Basically, the Intersection Observer API triggers a callback function in situations such as:

- When the position of the selected element comes into the client’s viewport.
- When a selected element intersects a parent/root element.
- When the observer is initially declared
Specification and Browser Compatibility.

As at the time of writing this article, the specifications were still a working draft. However, updates can be found here.
As for the browser compatibility, here is the current report:


![](https://paper-attachments.dropbox.com/s_27F54750736534E986D2E60F6C3491B4AFF8491308890171E1134DE46933F221_1632991834237_broswer_compatibility.png)



Applications of Intersection Observer.

Intersection Observer has been known for various applications such as:

- Optional rendering of DOM elements.
- Lazy Loading of contents.
- Loading Contents on demand of ‘infinite scrolling’ sites.
- Advertisement and animations rendering.
- Carousels
Using the Intersection Observer.

Firstly, what we will want to find out is if the browser supports the Intersection Observer API.
We can write a condition to check:


    if ('IntersectionObserver' in window) {
        console.log("IntersectionObserver is supported!");
    } else {
        console.log("IntersectionObserver is not supported!");
    }

The `ItersectionObserver` object is usually structure like this:


    let options= {
        root: null,
        rootMargin: '0px',
        threshold: 0.5
    };
    
    const callback = (entries){ // entries = array of targeted elements
        entries.forEach(entry=>{
            // what happens each entry
        })
    }
    
    let observerObj = new IntersectionObserver(callback, options);
    observerObj.observe();

So the `IntersectionObserver` object accepts two arguments:

- The `callback` function.
- Optional `options` object.  
The Intersection Observer Callback Function.

When the `callback` function is executed, each entry as a targeted element in a list of entries has certain properties that will determine what happens to the entry. Examples of those properties are:

- boundingClientRect
- intersectionRatio
- intersectionRect
- isIntersecting
- rootBounds
- target
- time

In this article, we will just be using isIntersecting to check if the current entry is intersecting with the root.

The Intersection Observer Options.

The `options` object contains properties that affect the outcome of a `callback` function. It contains the following properties:

- `root` by default or if null is the browser’s viewport. If an element is specified, it has to be a parent to the target.
- `rootMargin` by default is zero else, it can be valued as CSS margin. This gives the parent/root element some margin before detecting intersection.
- `threshold` can be either a number or an array of numbers. It represents to what percentage the target element should intersect the parent/root before the callback function is executed. The accepted values range from 0 to 1. If it is 0 then, it means the slightest pixel of the target element needs to intersect with its parent/root element, if it is 0.5, 50% of the target element needs to intersect with its parent/root else, if the value is 1, then 100% of the target element needs to intersect with its parent/root element for the callback function to be executed.
Targeting an Element to be Observed.


    /*
    In VanillaJs we can use querySelector to select a DOM element like this...
    */
    let targetElement = document.querySelector('#item')
    observerObj.observe(targetElement)
    
    //In ReactJs we can use the useRef hook like this...
    let targetRef = useRef(null); //Set a component to be ref of targetRef
    let targetElement = targetRef.current
    observerObj.observe(targetElement)
Creating an Intersection Observer Custom Hook in React


    import { useEffect, useMemo, useState } from 'react'
    const useElementOnScreen = (options, targetRef) => {
        const [isVisibile, setIsVisible] = useState()
        const callbackFunction = entries => {
            const [entry] = entries //const entry = entries[0]
            setIsVisible(entry.isIntersecting)
        }
        const optionsMemo = useMemo(() => {
            return options
        }, [options])
        useEffect(() => {
            const observer = new IntersectionObserver(callbackFunction, optionsMemo)
            const currentTarget = targetRef.current
            if (currentTarget) observer.observe(currentTarget)
            
            return () => {
            if(currentTarget) observer.unobserve(currentTarget)
            }
        }, [targetRef, optionsMemo])
        return isVisibile
    }
    export default useElementOnScreen 

Here, we are accepting `options` and `targetRef` as props then, we set a default state for the element where its visibility is null/false.
Inside our callback function, we are setting the `isVisible` state to the value returned when checking if the entry `isIntersecting`(we are always expecting true or false).
After observing the target element, we return the `isVisible` state.

Using the Intersection Observer Custom Hook In Our TikTok Application
Setting Up the Application. 

Not to waste time, I have created a starter project. It is available on my GitHub repository. 
To start the application running, open your terminal to a new work folder and run the following commands:

    git clone https://github.com/wolz-CODElife/Tiktok-clone.git


    cd Tiktok-clone


    npm install

In the folder downloaded, the following files and directories are present:


![](https://paper-attachments.dropbox.com/s_27F54750736534E986D2E60F6C3491B4AFF8491308890171E1134DE46933F221_1633234797556_files.png)


The files and folders we are working with are inside the `src`. As shown above we already have the custom hook that tells us when an element is visible or not.
So we will update the `Video.js` component to play/stop a video depending on its visibility status.

Inside the `Video.js` file, put the following code:


    import React, { useEffect, useRef, useState } from "react";
    import "./Video.css";
    import VideoFooter from "./VideoFooter";
    import VideoSidebar from "./VideoSidebar";
    import useElementOnScreen from './hooks/useElementOnScreen'
    import VideoPlayButton from "./VideoPlayButton";
    const Video = ({ url, channel, description, song, likes, messages, shares }) => {
      const [playing, setPlaying] = useState(false);
      const videoRef = useRef(null);
      const options = {
          root: null,
          rootMargin: '0px',
          threshold: 0.3
      }
      const isVisibile = useElementOnScreen(options, videoRef)
      const onVideoClick = () => {
        if (playing) {
          videoRef.current.pause();
          setPlaying(!playing);
        } else {
          videoRef.current.play();
          setPlaying(!playing);
        }
      };
      useEffect(() => {
        if (isVisibile) {
          if (!playing) {        
            videoRef.current.play();
            setPlaying(true)
          }
        }
        else {
          if (playing) {        
            videoRef.current.pause();
            setPlaying(false)
          }
        }
      }, [isVisibile])
    
    
      return (
        <div className="video">
          <video className="video_player" loop preload="true" ref={videoRef} onClick={onVideoClick} src={url}></video>
          <VideoFooter channel={channel} description={description} song={song} />
          <VideoSidebar likes={likes} messages={messages} shares={shares} />
          {!playing && <VideoPlayButton onVideoClick={onVideoClick} />}
        </div>
      );
    };
    export default Video;
    

What is happening here is: we imported the custom hook which is `useElementOnScreen`, then we use the value returned from the hook(which could be true or false) as the `isVisible` value.
Take note, we set the options for the Intersection Observer:

- root is `null` which means we are using the window as a parent element.
- rootMargin is `0px`.
- threshold is `0.3` which means once 30% of the target element is in the viewport, the callback function is triggered.

Then we use `UseEffect` to change the `playing` state of the video if the `isVisible` value changes.


    if (isVisibile) {
          if (!playing) {        
            videoRef.current.play();
            setPlaying(true)
          }
        }
        else {
          if (playing) {        
            videoRef.current.pause();
            setPlaying(false)
          }
        }

This code basically means if the video is visible, set the playing state to `true` if it is not yet playing, and if the video is not visible, set the playing state to `false`.

With this done, if we run the application using:

    npm start

If everything went well, we should have something like this:


![](https://paper-attachments.dropbox.com/s_27F54750736534E986D2E60F6C3491B4AFF8491308890171E1134DE46933F221_1633236840410_GIF-2021-10-03-05-51-59.gif)


If you wish to change the videos or even use a live database, edit the `video` state in `App.js`. Currently, we have this array of objects:


    [
        {
          url: 'https://res.cloudinary.com/codelife/video/upload/v1633232723/tiktok-clone/tiktok2_qxafx3.mp4',
          channel: 'DanceCrew',
          description: 'Video by Lara Jameson from Pexels',
          song: 'Bounce - Ruger',
          likes: 250,
          messages: 120,
          shares: 40
        },
        {
          url: 'https://res.cloudinary.com/codelife/video/upload/v1633232725/tiktok-clone/tiktok1_np37xq.mp4',
          channel: 'Happyfeet',
          description: '#happyfeetlegwork videos on TikTok',
          song: 'Kolo sound - Nathan',
          likes: 250,
          messages: 120,
          shares: 40
        },
        {
          url: 'https://res.cloudinary.com/codelife/video/upload/v1633232726/tiktok-clone/tiktok3_scmwvk.mp4',
          channel: 'thiskpee',
          description: 'The real big thug boys💛🦋 The real big thug boys💛🦋 ',
          song: 'original sound - KALEI KING 🦋',
          likes: 250,
          messages: 120,
          shares: 40
        },
      ]
Conclusion

Having created the application successfully, we should have learnt how Intersection Observer works and how you can implement the autoplay feature in TikTok, Instagram or Twitter with it.
With this knowledge, you can try implementing lazy loading images, carousels or even a infinite scrolling blog feeds page.
You can check the live demo of my TikTok clone here, I’ll advice viewing on a desktop browser for best experience. 
If you have any questions or remarks, please feel free to let me know in the comments or contact me for jobs and writing collaborations via Twitter @wolz_codelife.

