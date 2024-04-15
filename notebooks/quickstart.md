# Summarizing University Lectures with OpenAI

lectures suck
you can always find better information for a topic these days but what if you're actually taking a class for a grade?
You have to invest time into watching a teacher terribly explain something or briefly mentioning something that is actually rather important for you to study.

We can leverage OpenAI's technology to help us extract the information within any lecture.
This will allow anyone to venture out and learn through their preferred avenue of learning, while allowing the student to be aware of course specific information for a test.

Microsoft released a feature back in 2023 called [Intelligent Recap](https://techcommunity.microsoft.com/t5/microsoft-teams-blog/intelligent-meeting-recap-in-teams-premium-now-available/ba-p/3832541) which takes a **Microsoft Teams** meeting and summarizes it's contents. 
It leverages an speech-to-text and an LLM (probably OpenAI tools).
It would be interesting to stream a lecture through the tool to see how useful it is, given it's timestamp features. 
However, I (nor most students) don't have access to this tool, as it requires some sort of business account.

There are a couple of drawbacks to doing this with just a summary: no visual explanations and no timestamps to follow (was attempted but out of scope to do in a useful way).
In this article, I will explore the use of OpenAI's ChatGPT, and Whisper, along with langchain in order to summarize a lecture.
Ways to fill in the gaps of visual aids that are specific to the class will be addressed in the conclusion.



# Procedure

The procedure for this experiment goes as follows:

1. Download a lecture's audio
2. Send the audio to OpenAI's whisper
3. Create a Summary with ChatGPT

The fours steps are simple in principle, but each provide their own set of challenges to overcome.


## 1. Download a lecture's audio

I used the MIT recorded lecture for Quantum Mechanics titled ["Introduction to Superposition"](https://www.youtube.com/watch?v=lZ3bPUKo5zc) by Professor Allan Adams.
The video was downloaded, then converted to audio using he code below.


```python
from moviepy.editor import VideoFileClip

def vid2aud(video_path, audio_path, overwrite=False):
    if not video_path.endswith(".mp4"):
        raise ValueError("Only mp4 files are supported")
    if not audio_path.endswith(".mp3"):
        raise ValueError("Only mp3 files are supported")
    if not os.path.isfile(video_path):
        raise FileNotFoundError("Video file not found")
    if os.path.isfile(audio_path) and not overwrite:
        raise FileExistsError("Audio file already exists")
    
    video = VideoFileClip(video_path)
    audio = video.audio
    audio.write_audiofile(audio_path, bitrate='42k')  # Set the desired bitrate (e.g., '64k')

video_path = "..\data\Lecture 1 Introduction to Superposition.mp4"
audio_path = "..\data\output_audio.mp3"
vid2aud(video_path, audio_path, overwrite=True)

```

    MoviePy - Writing audio in ..\data\output_audio.mp3
    

                                                                              

    MoviePy - Done.
    

    


## 2. Send the Audio to OpenAI's Whisper

Details on interacting with OpenAI's API can be found [here](https://platform.openai.com/docs/api-reference).
This is pretty cut and dry.


```python

from dotenv import load_dotenv, find_dotenv
import os

load_dotenv(find_dotenv())


from openai import OpenAI
client = OpenAI()

audio_file= open(audio_path, "rb")
transcription = client.audio.transcriptions.create(
  model="whisper-1", 
  file=audio_file
)

transcription_file = "..\data\output_transcription.txt"

with open(transcription_file, 'w') as file:
    file.write(transcription.text)

```

    > The following content is provided under a Creative Commons license. Your support will help MIT OpenCourseWare continue to offer high quality educational resources for free. To make a donation or to view additional materials from hundreds of MIT courses, visit MIT OpenCourseWare at ocw.mit.edu. Hi, everyone. Welcome to 8.04 for spring 2013. This is the fourth and presumably final time that I will be teaching this class, so I'm pretty excited about it. So my name is Alan Adams. I'll be lecturing the course. I'm an assistant professor in course eight. I study string theory and its applications to gravity, quantum gravity, and condensed matter physics. Quantum mechanics, this is a course in quantum mechanics. Quantum mechanics is my daily language. Quantum mechanics is my old friend. I met quantum mechanics 20 years ago. I just realized that last night. It was kind of depressing. So old friend. It's also my most powerful tool. So I'm pretty psyched about it. Our recitation instructors are Barton Zwiebach, and Matt Evans. Matt's new to the department, so welcome him. Hi. So he just started his faculty position, which is pretty awesome. And our TA is Paolo Glorioso. Paolo, are you here? Hey, there you are. OK, so he's a person to send all complaints to. So just out of curiosity, how many of you all are course eight? Oh, awesome. How many of you all are, I don't know, 18? Solid. Six?...

```python
transcription_file = "..\data\output_transcription.txt"

with open(transcription_file, 'r') as file:
    transcription_text = file.read()
```

## 3. Create a Summary with ChatGPT

In working on this problem, I came across some problems with simple prompts to ChatGPT.
The most important rule for consistency is of course being very specific with your request.
In the following blocks of code, I settled on this specific prompt to keep the article short and concise.
Even with a very specific request, you will see that content of the response is varied.
I will demonstrate this by prompting a few times and showing the difference in their outputs.


```python

from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
```


```python

llm = ChatOpenAI(model_name="gpt-3.5-turbo")

output_parser = StrOutputParser()
prompt = ChatPromptTemplate.from_messages([
    ("system", """You are world class note taker and will 
summarize the following text into a numbered list of unique concepts, 
and their explanations as worded by the lecturer, for us to review later."""),
    ("user", "{input}")
])

chain = prompt | llm | output_parser

for n in range(2):
    output = chain.invoke({"input": transcription_text})
    print(output)
    print("\n\n")
```

    1. **Introduction to Quantum Mechanics**: Quantum mechanics involves understanding the behavior of particles like electrons in a way that defies classical intuition.
    2. **Color and Hardness Boxes**: Experimental setup involving color and hardness boxes reveal that electrons can be in a superposition of states such as black or white, and hard or soft.
    3. **Results of Superposition Experiments**: Electrons in a superposition of states can behave unexpectedly, leading to phenomena like always coming out white in certain experiments.
    4. **Implications of Superposition**: The concept of superposition challenges classical notions of particles having fixed properties, leading to the need for new language and understanding in quantum mechanics.
    
    
    
    1. Quantum Mechanics: Quantum mechanics is a framework that describes the behavior of particles at the atomic and subatomic levels. It includes principles like superposition, where particles can exist in multiple states simultaneously.
    
    2. Superposition: An electron inside an apparatus can be in a superposition state, meaning it is neither definitively hard nor soft, but rather exists as a combination of both states. This concept challenges classical intuition and is a fundamental aspect of quantum mechanics.
    
    3. Experimental Results: Experiments involving electrons passing through color and hardness boxes revealed that electrons exhibit behaviors that cannot be explained by classical physics. They can be in superposition states, leading to unpredictable outcomes that defy classical logic.
    
    4. Implications of Superposition: Superposition implies that particles like electrons can exist in multiple states at the same time until measured, at which point they collapse into a single state. This idea forms the basis of quantum mechanics and challenges traditional notions of particle behavior.
    
    5. Development of Intuition: Understanding and developing intuition for quantum phenomena, such as superposition, is a key aspect of learning quantum mechanics. It requires a shift from classical thinking to embrace the unique behaviors of particles at the quantum level.
    
    
    
    

This course is about quantum mechanics, however, there are some concepts in our output that pertain to unique concepts of the *classroom setting* itself. 
We shouldn't have any issue with this, as we can just pick the course specific material out ourselves.
Sometimes the concept is mentioned, sometimes it is also explained.

Focusing on just the material pertaining to quantum mechanics, I'll compare and contrast the concepts touched on below.

| Concepts             | Response 1  | Response2
|---------             |------------|--------  
| Quantum Mechanics    | X          | X        
| Superposition        | X          | X        
| Uncertainty Principle|            |          
| Determinism          |            |          
| Locality             |            |          


For a long lecture, there are really only a few quantum mechanic specific concepts that are elaborated on but there may be some that I missed. 
Other definitions within this topic such as electrons, molecules, and detectors are assumed to be known anyways.
Also, their formatting is not consistent, which we will see later does come down to prompt engineering.
You can see both have picked up the same concepts but have missed these other definitions that are particularly important.
Resolving this could also come down to prompt engineering, but I propose something else that will give us more fine grained control over the process of parsing the knowledge in the lecture.

Let's revisit both the transcription step (step 2) and this one (step 3).

## 2. (improved) Send the Audio to OpenAI's Whisper

Here we will leverage another feature of Whisper. Time Stamps.
Segments are delimited by each sentence (I believe).
Now we can simply summarize groups of sentences to give us more fine grained information.




```python
from openai import OpenAI
client = OpenAI()

audio_path = "..\data\output_audio.mp3"

audio_file= open(audio_path, "rb")
transcription = client.audio.transcriptions.create(
  model="whisper-1", 
  response_format="verbose_json",
  timestamp_granularities=["segment"],
  file=audio_file
)
print(transcription.text)


transcription_file = "..\data\output_transcription.json"

import json
with open(transcription_file, 'w') as file:
    json.dump(transcription.segments, file)

transcription_segments = transcription.segments

```

    The following content is provided under a Creative Commons license. Your support will help MIT OpenCourseWare continue to offer high quality educational resources for free. To make a donation or to view additional materials from hundreds of MIT courses, visit MIT OpenCourseWare at ocw.mit.edu. Hi, everyone. Welcome to 8.04 for spring 2013. This is the fourth and presumably final time that I will be teaching this class, so I'm pretty excited about it. So my name is Alan Adams. I'll be lecturing the course. I'm an assistant professor in course eight. I study string theory and its applications to gravity, quantum gravity, and condensed matter physics. Quantum mechanics, this is a course in quantum mechanics. Quantum mechanics is my daily language. Quantum mechanics is my old friend. I met quantum mechanics 20 years ago. I just realized that last night. It was kind of depressing. So old friend. It's also my most powerful tool. So I'm pretty psyched about it. Our recitation instructors are Barton Zwiebach, and Matt Evans. Matt's new to the department, so welcome him. Hi. So he just started his faculty position, which is pretty awesome. And our TA is Paolo Glorioso. Paolo, are you here? Hey, there you are. OK, so he's a person to send all complaints to. So just out of curiosity, how many of you all are course eight? Oh, awesome. How many of you all are, I don't know, 18? Solid. Six? Excellent. Nine. No one? This is the first year we haven't had anyone course nine. That's a shame. So last year, one of the best students was a course nine student. So two practical things to note. The first thing is everything that we put out will be on the Stellar website. Lecture notes, homeworks, exams, everything is going to be done through Stellar, including your grades. The second thing is that, as you may notice, there are rather more lights than usual. I'm wearing a mic, and there are these signs up. We're going to be videotaping this course for the lectures for OCW. And if you don't want to, if you're happy with that, cool. If not, just sit on the sides, and you won't appear anywhere on video. Sadly, I can't do that. But you're welcome to if you like. But hopefully that should not play a meaningful role in any of the lectures. So I'm going to, OK. So the goal of 804 is for you to learn quantum mechanics. And by learn quantum mechanics, I don't mean learn how to do calculations, although that's an important and critical thing. I mean learn some intuition. I want you to develop some intuition for quantum phenomena. Now, quantum mechanics is not hard. It has a reputation as being a hard topic. It is not a super hard topic. It does, however, require, so in particular, everyone in this room, I'm totally positive, can learn quantum mechanics. It does require concerted effort. It's not a trivial topic. And in order to really develop a good intuition, the essential thing is to solve problems. So the way you develop a new intuition is by solving problems and by dealing with new situations, new contexts, new regimes, which is what we're going to do in 804. It's essential that you work hard on the problem sets. So your job is to devote yourself to the problem sets. My job is to convince you at the end of every lecture that the most interesting thing you could possibly do when you leave is the problem set. So you decide who has the harder job. So the workload is not so bad. So we have problem sets due. They're due in the physics box in the usual places by lecture, by 11 AM sharp on Tuesdays, every week. Late work, no, not so much. But we will drop one problem set to make up for unanticipated events. We'll return the graded problem sets a week later in recitation. Should be easy. I strongly, strongly encourage you to collaborate with other students on your problem sets. You will learn more. They will learn more. It will be more efficient. Work together. However, write up your problem set yourself. That's the best way for you to develop and test your understanding. There will be two midterms, dates to be announced, and one final. I guess we could have multiple, but that would be a little exciting. We're going to use clickers, and clickers will be required. We're not going to take attendance, but they will give a small contribution to your overall grade. And we'll use them, most importantly, for non-graded, but just participation, concept questions, and the occasional in-class quiz to probe your knowledge. This is mostly so that you have a real-time measure of your own conceptual understanding of the material. This has been enormously valuable. And something I want to say just right off is that the way I've organized this class is not so much based on how the classes I was taught. It's based, to the degree possible, on empirical lessons about what works in teaching, what actually makes you learn better. And clickers are an excellent example of that. So this is mostly a standard lecture course, but there will be clickers used. So by next week, I need you all to have clickers, and I need you all to register them on the TSG website. So I haven't chosen a specific textbook, and this is discussed on the Stellar web page. There are a set of textbooks, four textbooks that I strongly recommend, and a set of others that are nice references. The reason for this is twofold. First off, there are kind of two languages that are canonically used for quantum mechanics. One is called wave mechanics, and the mathematical language is partial differential equations. The other is matrix mechanics. They have big names. And the language there is linear algebra. And different books emphasize different aspects and use different languages. And they also try to aim at different problems. Some books are aimed towards people who are interested in material science. Some books are aimed towards people interested in philosophy. And depending on what you want, get the book that's suited to you. And every week, I'll be providing, with your problem sets, readings from each of the recommended texts. So what I really encourage you to do is find a group of people to work with every week, and make sure that you've got all the books covered between you. This will give you as much access to the text as possible without forcing you to buy four books, which I would discourage you from doing. So finally, I guess the last thing to say is, if this stuff were totally trivial, you wouldn't need to be here. So ask questions. If you're confused about something, lots of other people in the class are also going to be confused. And if I'm not answering your question without you asking, then no one's getting the point. So ask questions. Don't hesitate to interrupt. Just raise your hand, and I will do my best to call on you. And this is true both in lecture. Also, go to office hours and recitations. Ask questions. I promise, there's no such thing as a terrible question. Someone else will also be confused. So it's very valuable to me and everyone else. OK. So before I get going on the actual physics content of the class, are there any other practical questions? Yeah? AUDIENCE MEMBER 2. What did you say was a lateness policy?  Lateness policy. No late work is accepted whatsoever. AUDIENCE MEMBER 3. So you're saying there's like a make out for it? PROFESSOR 1. So the deal is, given that every once in a while, you're going to be walking to school, and your leg is going to fall off, and your dog's going to jump out and eat your person standing next to you, whatever, things happen. So we will drop your lowest problem set score without any questions. At the end of the semester, we'll just drop your lowest score. And if you turn them all in, great. Whatever your lowest score was, fine. If you missed one, then gone. On the other hand, if you know like next week, I'm going to be attacked by a rabid squirrel, it's going to be horrible. I don't want to have to worry about my problem set. Could we work this out? So if you know ahead of time, come to us. But you need to do that well ahead of time, the night before doesn't count. OK? Yeah? AUDIENCE 2. Will we be able to watch the videos? PROFESSOR 1. That's a good question. I don't know. I don't think so. I think it's going to happen at the end of the semester. Yeah, OK, so no. You'll be able to watch them later on the OCW website. Other questions? Yeah? AUDIENCE 3. Do you have any other videos that you recommend, like other courses on YouTube? PROFESSOR 1. That's an interesting question. I don't know off the top of my head, but if you send me an email, I'll preserve it. Because I do know several other lecture series that I like very much, but I don't know if they're available on YouTube or publicly. So send me an email and I'll check. Yeah? AUDIENCE 4. How about the reading assignments? PROFESSOR 1. Reading assignments on the problem set every week will be listed. There will be equivalent reading from every textbook. And if there is something missing, like if no textbook covers something, I'll post a separate reading. Every once in a while, I'll post auxiliary readings, and they'll be available on the Stellar website. So for example, on your problem set, which is the first one was posted, will be available immediately after a lecture on the Stellar website. There are three papers that it refers to, or two. And they're posted on the Stellar website and linked from the problem set. Cool. Others? OK. So the first lecture, the content of the physics of the first lecture is relatively standalone. It's going to be an introduction to a basic idea that is going to haunt, plague, and charm us through the rest of the semester. OK? The logic of this lecture is based on a very beautiful discussion in the first few chapters of a book by David Albert called Quantum Mechanics and Experience. It's a book for philosophers, but the first few chapters are really lovely introduction at a non-technical level. And I encourage you to take a look at them, because they're very lovely. But it's, to be sure, straight up physics. OK. Ready? I love this stuff. So today, I want to describe to you a particular set of experiments. Now to my mind, these are the most unsettling experiments ever done. These experiments involve electrons. They have been performed. And the results, as I will describe them, are true. I'm going to focus on two properties of electrons. I will call them color and hardness. OK? And these are not the technical names. We'll learn the technical names for these properties later on in the semester. But to avoid distracting you by preconceived notions of what these things mean, I'm going to use ambiguous labels, color and hardness. And the empirical fact is that every electron that's ever been observed is either black or white and no other color. We've never seen a blue electron. There are no green electrons. No one has ever found a fluorescent electron. They're either black or they are white. It is a binary property. Secondly, their hardness is either hard or soft. They are never squishy. No one's ever found one that dribbles. They're either hard or they are soft. Binary properties. OK? Now what I mean by this is that it is possible to build a device which measures the color and the hardness. In particular, it is possible to build a box, which I will call a color box, that measures the color. And the way it works is this. It has three apertures, an in port and two out ports, one which sends out black electrons and one which sends out white electrons. And the utility of this box is that the color can be inferred from the position. If you find the electron over here, it is a white electron. If you find the electron here, it is a black electron. Cool? Similarly, we can build a hardness box, which again has three apertures, an in port, and hard electrons come out this port and soft electrons come out this port. OK? Now if you want, you're free to imagine that these boxes are built by putting a monkey inside. And you send in an electron, and the monkey, with the ears, looks at the electron and says, it's a hard electron that sends it out one way or it's a soft electron that sends out the other. The workings inside do not matter. And in particular, later in the semester, I will describe in considerable detail the workings inside this apparatus. And here's something I want to emphasize to you. It can be built, in principle, using monkeys, hyperintelligent monkeys that can see electrons. It could also be built using magnets and silver atoms. It could be done with neutrons. It could be done with all sorts of different technologies. And they all give precisely the same results as I'm about to describe. OK? They all give precisely the same results. So it does not matter what's inside. But if you want a little idea, you can imagine putting a monkey inside, a hyperintelligent monkey. I don't know, it sounds good. So I'm going to call it, yeah. So a key property of these hardness boxes and color boxes is that they are repeatable. And here's what I mean by that. If I send in an electron and I find that it comes out of a color box, black, and then I send it in again, then if I send it into another color box, it comes out black again. So in diagrams, if I send in some random electron to a color box and I discover that it comes out, let's say, the white aperture, and so here's dot dot dot, and I take the ones that come out the white aperture and I send them into a color box again, then with 100% confidence, 100% of the time, the electron coming out of the white incident on the color box will come out the white aperture again. And 0% of the time will it come out the black aperture. So this is a persistent property. You notice that it's white. You measure it again, it's still white. Do it a little bit later, it's still white. It's a persistent property. Ditto the hardness. If I send in a bunch of electrons into a hardness box, here's an important thing. Well, send them into a hardness box, and I take out the ones that come out soft, and I send them again into a hardness box, and they come out soft, they will come out soft with 100% confidence, 100% of the time. Never do they come out the hard aperture. Any questions at this point? So here's a natural question. Might the color and the hardness of an electron be related? And more precisely, might they be correlated? Might knowing the color infer something about the hardness? So for example, so being male and being a background being male and being a bachelor are correlated properties. Because if you're male, you don't know if you're a bachelor or not. But if you're a bachelor, you're male. That's the definition of the word. So is it possible that color and hardness are similarly correlated? So I don't know. There are lots of good examples. Wearing a red shirt and beaming down to the surface and making it back to the enterprise later after the away team returns. Correlated, right? Negatively, but correlated. So the question is, suppose, e.g., suppose we know that an electron is white, does that determine the hardness? So we can answer this question by using our boxes. So here's what I'm going to do. I'm going to take some random set of electrons. That's not random. Random. And I'm going to send them into a color box. And I'm going to take the electrons that come out the white aperture. And here's a useful fact. When I say random, here's operationally what I mean. I take some piece of material, I scrape it, I pull off some electrons, and they're totally randomly chosen from the material, and I send them in. If I send a random pile of electrons into a color box, useful thing to know, they come out about half and half. It's just some random assortment. Some of them are white, some of them come out black. Suppose I send some random collection of electrons into a color box, and I take those which come out the white aperture. And I want to know, does white determine hardness? So I can do that, check, by then sending these white electrons into a hardness box and seeing what comes out. And what we find is that 50% of those electrons incident on the hardness box come out hard. And 50% come out soft. And ditto if we reverse this. If we take hardness and take, for example, a soft electron and send it into a color box, we again get 50-50. So if you take a white electron, you send it in a hardness box, you're at even odds. You're at chance as to whether it's going to come out hard or soft. And similarly, if you send a soft electron into a color box, even odds it's going to come out black or white. So knowing the hardness does not give you any information about the color, and knowing the color does not give you any information about the hardness. Cool? These are independent facts, independent properties. They're not correlated in this sense, in precisely this operational sense. Cool? Questions? OK. So measuring the color gives zero predictive power for the hardness, and measuring the hardness gives zero predictive power for the color. And from that, I will say that these properties are uncorrelated. So H, hardness, and color are, in this sense, uncorrelated. Cool? So using these properties of the color and hardness boxes, I want to run a few more experiments. I want to probe these properties of color and hardness a little more. And in particular, knowing these results allows us to make predictions to predict the results for a set of very simple experiments. Now, what we're going to do for the next bit is we're going to run some simple experiments, and we're going to make predictions. And then those simple experiments are going to lead us to more complicated experiments. But let's make sure we understand the simple ones first. So for example, let's take this last experiment, color hardness, and let's add a color box. One more monkey. So color in, and we take those that come out the white aperture, and we send them into a hardness box. Hard, soft. And we take those electrons which come out the soft aperture, and now let's send these, again, into a color box. So it's easy to see what to predict. Black, white. So you can imagine a monkey inside this going, aha. You look at it, you inspect, it comes out white. Here, you look at it, inspect, it comes out soft. And you send it into the color box. And what do you expect to happen? Well, let's think about the logic here. Anything reaching the hardness box must have been measured to be white. And we just did the experiment that if you send a white electron into a hardness box, 50% of the time it comes out a hard aperture, and 50% of the time it comes out the soft aperture. So now we take that 50% of electrons that comes out the soft aperture, which have previously been observed to be white and soft, and then we send them into a color box, and what happens? Well, since color is repeatable, the natural expectation is that, of course, it comes out white. So our prediction, a natural prediction here, is that of those electrons that are incident on this color box, 100% should come out white. And 0% should come out black. Does that seem like a reasonable? Let's just make sure that we're all agreeing. Let's vote. How many people think this is probably correct? OK, good. How many people think this is probably wrong? OK, good. That's reassuring. Except you're all wrong, right? In fact, what happens is half of these electrons exit white, 50%. And 50% exit black. So let's think about what's going on here. This is really kind of troubling. We've said already that knowing the color doesn't predict the hardness. And yet, this electron, which was previously measured to be white, now when subsequently measured, sometimes it comes out white, sometimes it comes out black, 50% to 50% of the time. So that's surprising. What that tells you is you can't think of an electron as a little ball that has black and soft written on it, right? You can't, because apparently that black and soft isn't a persistent thing. Although, it's persistent in the sense that once it's black, it stays black. So what's going on here? Now, I should emphasize that the same thing happens if I had changed this to taking the black electrons and throwing in hardness and picking soft and then measure the color, or if I had used the hard electrons. Any of those combinations, any of these ports, would have given the same results 50-50. It is not persistent in this sense. Apparently, the presence of the hardness box tampers with the color somehow. So it's not quite as trivial as that hyper-intelligent monkey. Something else is going on in here. So this is suspicious. So here's the first natural move. The first natural move is, oh, look, surely there's some additional property of the electron that we just haven't measured yet that determines whether it comes out the second color box black or white. There's got to be some property that determines this. And so people have spent a tremendous amount of time and energy looking at these initial electrons and looking with great care to see whether there's any sort of feature of these incident electrons which determines which port they come out of. And the shocker is, no one's ever found such a property. No one has ever found a property which determines which port it comes out of. As far as we can tell, it is completely random. Those that flip and those that don't are indistinguishable at the beginning. And let me just emphasize, if anyone found such a, it's not like we're not looking. If anyone found such a property, fame, notoriety, subverting quantum mechanics, Nobel Prize, people have looked. And there is none that anyone's been able to find. And as we'll see later on, using Bell's inequality, we can more or less nail that such things don't exist, such a feature doesn't exist. But this tells us something really disturbing. This tells us, and this is the first real shocker, that there is something intrinsically unpredictable, non-deterministic, and random about physical processes that we observe in a laboratory. There is no way to determine a priori whether it will come out black or white from the second box. Probability in this experiment, it's forced upon us by observations. OK, well, there's another way to come at this. You could say, look, you ran this experiment, that's fine. But look, I've met the guy who built these boxes. And look, he's just some guy, right? And he just didn't do a very good job, right? The boxes are just badly built. So here's the way to defeat that argument. No, we've built these things out of different materials, using different technologies, using electrons, using neutrons, using buckyballs, C60. Seriously, it's been done, OK? We've done this experiment, and this property does not change. It is persistent. And the thing that's most upsetting to me is that not only do we get the same results, independent of what objects we use to run the experiment, we cannot change the probability away from 50-50 at all. To within experimental tolerances, we cannot change, no matter how we build the boxes, we cannot change the probability by part in 100, OK? 50-50. And to anyone who grew up with determinism from Newton, this should hurt. This should feel wrong. But it's a property of the real world, and our job is going to be to deal with it, OK? Rather, your job is going to be to deal with it, because I went through this already, so. OK? So here's a curious consequence. Oh, any questions before I cruise? OK. So here's a curious consequence of this series of experiments. Here's something you can't do. Are you guys old enough where you can't do this on television? Oh, Jesus. This is so sad. OK. So here's something you can't do. We cannot build, it is impossible to build, a reliable color and hardness box. We've built a box that tells you what color it is. We've built a box that tells you what hardness it is. But you cannot build a meaningful box that tells you what color and hardness an electron is. So in particular, what would this magical box be? It would have four ports. And its ports would say, well, one is white and hard, and one is white and soft, and one is black and hard, and one is black and soft. So you can imagine how you might try to build a color and hardness box. So for example, here's something that you might imagine. Take your incident electrons, and first send them into a color box, OK? And take those white electrons, and send them into a hardness box. And take those electrons, and this is going to be white and hard, and this is going to be white and soft, OK? And similarly, send these black electrons into a hardness box. And here's hard and black, and here's soft and black, OK? Everyone cool with that? So this seems to do the thing I wanted. It measures both the hardness and the color. What's the problem with it? Yeah, exactly. So the color is not persistent. So you tell me this is a soft and black electron, right? That's what you told me. Here's the box. But if I put a color box here, that's the experiment we just ran. And what happens? Does this come out black? No, this is a crappy source of black electrons. It's 50-50 black and white, right? So this box can't be built. And the reason, and I want to emphasize this, the reason we cannot build this box is not because our experiments are crude. And it's not because I can't build things, although that's true. I was banned from a lab one day after joining it, actually. So I really can't build. But other people can, OK? And that's not why. We can't because there's something much more fundamental, something deeper, something in principle, which is encoded in this awesome experiment, OK? This can't be done. It does not mean anything as a consequence. It does not mean anything to say this electron is white and hard. Because if you tell me it's white and hard and I measure white, well, I know that if it's hard, it's going to come out 50-50. It does not mean anything. So this is an important idea. This is an idea which is enshrined in physics with a term which comes with capital letters, the uncertainty principle. And the uncertainty principle says basically that, look, there's some observable, measurable properties of a system which are incompatible with each other in precisely this way. Incompatible with each other in the sense that not that you can't know because you can't know whether it's hard and soft simultaneously. Deeper. It is not hard and white simultaneously. It cannot be. It does not mean anything to say it is hard and white simultaneously. That is uncertainty. And again, uncertainty is an idea we are going to come back to over and over in the class. But every time you think about it, this should be the first place you start for the next few weeks. Yeah. Questions? No questions? OK. So at this point, it's really tempting to think, yeah, OK, this is just, you know, this is just about the hardness and the color of electrons. Um, you know, it's just a weird thing about electrons, not a weird thing about the rest of the world. The rest of the world is completely reasonable. And no, that's absolutely wrong. Every object in the world has the same properties. If you take buckyballs and you send them through the analogous experiment, I will show you the data, I think tomorrow, but soon. I will show you the data when you take buckyballs and run it through a similar experiment, you get the same effect. Now, buckyballs are huge, right? 60 carbon atoms. But OK, OK, at that point, you're saying, like, dude, come on, huge. 60 carbon atoms. So there is a pendulum, depending on how you define building, in this building, a pendulum, which is used in principle, which is used to improve detectors to detect gravitational waves. There's a pendulum with a, I think it's 20 kilo mirror, and that pendulum exhibits the same sort of effects here. We can see these quantum mechanical effects in those mirrors. And this is in breathtakingly awesome experiments done by Nergis Malvalavala, whose name I can never pronounce, but who is totally awesome. She's an amazing physicist. And she can get these kind of quantum effects out of a 20 kilo mirror. So before you say something silly, like, ah, it's just electrons, it's 20 kilo mirrors. And if I could put you on a pendulum that accurate, it would be you. These are properties of everything around you. The miracle is not that electrons behave oddly. The miracle is that when you take 10 to the 27 electrons, they behave like cheese. That's the miracle, OK? This is the underlying correct thing. OK, so this is so far so good. But let's go deeper. Let's push it. And to push it, I want to design for you a slightly more elaborate apparatus, a slightly more elaborate experimental apparatus. And for this, I want you to consider the following device. I'm going to need to introduce a couple of new features for you. Here's a hardness box. And it has an in port. And the hardness box has a hard aperture. And it has a soft aperture. And now, in addition to this hardness box, I'm going to introduce two elements. First, mirrors, OK? And what these mirrors do is they take the incident electrons and nothing else, they change the direction of motion, OK? Change the direction of motion. And here's what I mean by doing nothing else. If I take one of these mirrors, and I take, for example, a color box, and I take the white electrons that come out, and I bounce it off the mirror, OK, and then I send these into a color box, then they come out white 100% of the time. It does not change the observable color. Cool? All it does is change the direction. Similarly, with the hardness box, it doesn't change the hardness, it just changes the direction of motion, OK? And every experiment we've ever done on these guys changes in no way whatsoever the color or the hardness by subsequent measurements. Cool? Just changes the direction of motion. And then I'm going to add another mirror. It's actually a slightly fancy set of mirrors. All they do is they join these beams together into a single beam. And again, this doesn't change the color. You send in a white electron, you get out, and you measure the color on the other side, you get a white electron. You send in a black electron from here, and you measure the color, you get a black electron again out. Cool? So here's my apparatus. And I'm going to put this inside a big box. And I want to run some experiments with this apparatus. Everyone cool with the basic design? Any questions before I cruise on? This part is fun. OK. The first, so what I want to do now is I want to run some simple experiments before we get to fancy stuff. And the simple experiments are just going to warm you up, OK? They're going to prepare you to make some predictions and some calculations. And eventually, we'd like to lead back to this guy. So the first experiment, I'm going to send in white electrons. Whoops, eem, eem, eem, eem. I'm going to send in white electrons. And I'm going to measure at the end, and in particular, at the output, the hardness. OK? So I'm going to send in white electrons. And I'm going to measure the hardness. So this is my apparatus. I'm going to measure the hardness at the output. And what I mean by measure the hardness is I throw these electrons into a hardness box and see what comes out. OK, so this is experiment one. And let me draw this. Let me embiggen the diagram. So you send white into. So the mechanism is a hardness box, mirror, mirror, mirrors. And now, whoops, we're measuring the hardness out, so hardness. And the question I want to ask is, how many electrons come out the hard aperture, and how many electrons come out the soft aperture of this final hardness box? OK? So I'd like to know what fraction come out hard and what fraction come out soft. I send an initial white electron. For example, I took a color box and took the white output. Send them into the hardness box, mirror, mirror, hard, hard, soft. And what fraction come out hard and what fraction come out soft? OK, so just think about it for a minute. OK. And when you have a prediction in your head, raise your hand. All right, good. Walk me through your prediction. 50-50. How come? I'm seeing four different colors, and how many vary on hardness. Good. So it's the same with the hardness box, every step from there. And then, once again, when you go to the other hardness box, it's the same with the hardness box. Awesome. So let me say that again. So we've done the experiment. You send a white electron into a hardness box, and we know that it's non-predictive, 50-50. So if you take a white electron and you send it into the hardness box, 50% of the time it will come out the hard aperture, and 50% of the time it will come out the soft aperture. Now, if you take the one that comes out the hard aperture, then you send it up here, send it up here. We know that these mirrors do nothing to the hardness of the electron except change the direction of motion. We've already done that experiment. So you measure the hardness at the output, what do you get? Hard, because it came out hard. Mirror, mirror, hardness, hard. But it only came out hard 50% of the time, because we sent in an initially white electron. Yeah? What about the other 50%? Well, the other 50% of the time it comes out the soft aperture and follows what I'll call the soft path to the mirror, mirror, hardness. And with soft, mirror, mirror, hardness, you know it comes out soft. 50% of the time it comes out this way, and then it will come out hard. 50% it follows a soft path, and then it will come out soft. Was this the logic? Good. How many people agree with this? Solid. How many people disagree? No abstention. OK. So here's a prediction. Oh, yep? Just a question. Could you justify that prediction without talking about, oh, well, half the electrons were initially measured to be hard, and half were initially measured to be soft, by just saying, well, we have a hardness box, and then we join these electrons together again. So we don't know anything about it. So it's just like sending white electrons into one hardness box instead of two. Yeah, that's a really tempting argument, isn't it? Let's see. We're going to see in a few minutes whether that kind of an argument is reliable or not. But so far, we've been given two different arguments that lead to the same prediction, 50-50. Yeah? Question? Those, like, the electrons interact between themselves? Like, when you get them together. Yeah. This is a very good question. So here's a question. Look, you're sending a bunch of electrons into this apparatus. But if I take two, look, I took 802, you know, you take two electrons, you put them close to each other, what do they do? Right? They interact with each other through a potential. Right? So we're being a little bold here, throwing a bunch of electrons in, saying, oh, they're independent. So I'm going to do one better. I will send them in one at a time. One electron through the apparatus. And then it will wait for six weeks. And then I'll say, see, you guys laugh, you think that's funny, but there's a famous story of a guy who did a similar experiment with photons, OK, French guy, and I mean the French, they know what they're doing. So he wanted to do the same experiment with photons. But the problem is, if you take a laser and you shine it into your apparatus, then there are like 10 to the 18 photons in there at any given moment. And the photons, who knows what they're doing with each other? Right? So I want to send in one photon, but the problem is it's very hard to get a single photon. Very hard. So what he did, did you not, he took an opaque barrier. I don't remember what it was, it was some sort of film on top of glass, I think it was some sort of oil tar film. Barton, do you remember what he used? OK, so he takes a film and it has this opaque property such that the photons that are incident upon it get absorbed. Once in a blue moon, a photon manages to make its way through. Literally, like once every couple of days, OK? Now, or a couple of hours, I think. So it's going to take a long time to get any sort of statistics. But he has this advantage that once every couple of hours or whatever, a photon makes its way through. That means inside the apparatus, if it takes a picosecond to cross, triumph. Right? That's the week I was talking about. So he does this experiment, but as you can tell, you start the experiment, you press go, and then you wait for six months. Side note on this guy. Liked boats. Really liked yachts. So he had six months to wait before doing a beautiful experiment and having the results. And what did he do? Went on a world tour in his yacht. Comes back, collects the data, and declares victory because, indeed, he saw the effect he wanted. So I was not kidding. So we really do wait. So I will take your challenge and single electron, throw it in, let it go through the apparatus, takes mere moments, wait for a week, send in another electron. No electrons are interacting with each other. OK? Just a single electron at a time going through this apparatus. Other complaints? Sorry? Oh, you'll get them. I have a hard time resisting. So here's the prediction. 50-50. We now have two arguments for this. So again, let's vote after the second argument. 50-50, how many people? You sure? Positive? OK. How many people don't think so? Very small dust. OK. It's correct. Yay! OK. So good. So I like messing with you guys. So remember, we're going to go through a few experiments first, where it's going to be very easy to predict the results. We've got four experiments like this to do. And then we'll go on to the interesting examples. But we need to go through them so we know what happens, so we can make an empirical argument rather than an in-principle argument. So there's the first experiment. Now I want to run the second experiment. And the second experiment, same as the first, a little bit louder, a little bit worse. Sorry. The second experiment, we're going to send in hard electrons. And we're going to measure color at out. So again, let's look at the apparatus. We send in hard electrons. And our apparatus is hard box, hardness box, with a hard and a soft aperture. And now we're going to measure the color at the output. Color. What have I been doing? And now I want to know, what fraction come out black and what fraction come out white? We're using lots of monkeys in this process. OK, so this is not rocket science. Rocket science isn't that complicated. Neuroscience is much harder. This is not neuroscience, OK? So let's figure out what this is. Predictions. So again, think about your prediction in your head. Come to a conclusion. Raise your hand when you have an idea. And just because you don't raise your hand doesn't mean I won't call on you. OK. 50-50 black and white. I like it. Tell me why. Great. So the statement, I'm going to say that slightly more slowly. That was an excellent argument. We have a hard electron. We know that hardness boxes are persistent. If you send a hard electron in, it comes out hard. So every electron incident upon our apparatus will transit across the hard trajectory. It will bounce, it will bounce, but it is still hard because we've already done that experiment. The mirrors do nothing to the hardness. So we send a hard electron into the color box, and what comes out? Well, we've done that experiment too. Hard into color, 50-50. So prediction is 50-50. This is your prediction. Is that correct? Awesome. OK. Let us vote. How many people think this is correct? Gusto. I like it. How many people think it's not? All right. Yay. This is correct. OK. Third experiment, slightly more complicated, but we have to go through these to get to the good stuff. So humor me for a moment. Third, let's send in white electrons and then measure the color at the output port. So now we send in white electrons. Same beast, and our apparatus is hardness box with a hard path and a soft path. Do-do-do, mirror, do-do-do, mirror, box, joined together into our out. And now we send those out electrons into a color box. OK? And our color box, black and white. And now the question is, how many come out black and how many come out white? OK? Again, think through the logic. Follow the electrons. Come up with a prediction. Raise your hand when you have a prediction. Yeah? How about earlier we showed that even, so we cut all the way to putting it into the hardness box. And so we know that it's good for us. So it'll take those paths equally. And then. With equal probability. Yeah. And then it'll go back inside the color box. But earlier when we did the same thing without the mirror's path changing, it came out as a good result. So I would say it's still good. Great. Let me say that again out loud. And tell me if this is an accurate sort of extension of what you said. I'm just going to use more words. But it's, I think, the same logic. We have a white electron, initially a white electron. We send it into a hardness box. When we send a white electron into a hardness box, we know what happens. 50% of the time it comes out hard, the hard aperture. 50% of the time it comes out the soft aperture. Consider those electrons that came out the hard aperture. Those electrons that came out the hard aperture will then transit across the system, preserving their hardness by virtue of the fact that these mirrors preserve hardness, and end up at a color box. When they end at the color box, when that electron, the single electron in the system, ends at this color box, then we know that a hard electron entering the color box comes out black or white 50% of the time. We've done that experiment too. So for those 50% that came out hard, we get 50-50. Now consider the other 50%. The other half of the time, a single electron in the system will come out the soft aperture. It will then proceed along the soft trajectory, bounce-bounce, not changing its hardness, and is then a soft electron incident on the color box. But we've also done that experiment, and we get 50-50 out black and white. So those electrons that came out hard come out 50-50, and those electrons that come out soft come out 50-50. And the logic then leads to 50-50 twice, 50-50. Was that an accurate statement? Good, it's a pretty reasonable extension. OK, let's vote. How many people agree with this one? OK, and how many people disagree? OK, so vast majority agree, and the answer is no. This is wrong. In fact, of these, 100% come out white, and 0 come out black. Never, ever does an electron come out the black aperture. I would like to quote what a student just said, because it's actually the next line in my notes, which is, what the hell is going on? So let's run a series of follow-up experiments to tease out what's going on here. OK, so something very strange. Let's just all agree something very strange just happened. OK, we sent a single electron in, and that single electron comes out the hardness box. Well, it either came out the hardness box or it came out the softness. It either came out the hard aperture or the soft aperture. And if it came out the hard, we know what happened. If it came out the soft, we know what happens. And it's not 50-50, right? So we need to improve the situation. Hold on a sec. Hold on one sec. So OK, go ahead. Yeah, I have a question about the setup. Yeah. So for the second hardness box, are we collecting both the soft and hard? The second? You mean the first hardness box? Or the one, the hardening? Which one? Sorry. The series one. This guy? Oh, that's a mirror, not a hardness box. So all this does, oh, thanks for asking. OK, yeah, sorry. I wish I had a better notation for this, but I don't. There's a classic, well, I'm not going to go into it. So remember that thing where I can't stop myself from telling stories? So all this does, it's just a set of mirrors. It's a set of fancy mirrors. And all it does is it takes an electron coming this way or an electron coming this way, and both of them get sent out in the same direction. It's like a beam joiner, right? It's like a y-junction. That's all it is. So if you will, imagine the box is a box, and you take, I don't know, Professor Zwiebach, and you put him inside. And every time an electron comes up this way, he throws it out that way. And every time it comes in this way, he throws it out that way. And he'd be really ticked at you for putting him in a box, but he'd do the job well. Yeah? And this also works if you go one electron at a time? This works if you go one electron at a time. This works if you go 14 electrons at a time. It works. It works reliably. Yeah? Just maybe we'll spend just one second on what's the difference between these experiments and that one? Yeah, I know. Right? Right? Yeah. So the question was, what's the difference between this experiment and the last one? Yeah, good question. So we're going to have to answer that. Yeah? Well, you're mixing, again, the hardness. So it's like as if you weren't measuring it at all, right? Apparently, it's a lot like we weren't measuring it, right? Because we send in the white electron. And at the end, we get out that it's still white. So somehow, this is like not doing anything. But how does that work? So that's an excellent observation. And I'm going to build you now a couple of experiments that tease out what's going on. And you're not going to like the answer. Yeah? What were the white electrons that were generated in this experiment? The white electrons were generated in the following way. I take a random source of electrons. I rub a cat against a balloon. And I charge up the balloon. OK. And so I take those random electrons. And I send them into a color box. And we have previously observed that if you take random electrons and throw them into a color box and pull out the electrons that come out the white aperture, if you then send them into a color box again, they're still white. So that's how I've generated them. I could have done it by rubbing the cat against glass or rubbing it against, you know, me, right, stroke the cat. But any randomly selected set of electrons sent into a color box, and then from which you take the white electrons. The electrons that didn't come out of the cat, right? Yeah. Uh-huh. Exactly. Yeah? Is there a difference that you never actually know about the electrons that you're sending? That's a really good question. So I don't want to, here's something I'm going to be very careful not to say in this class to the degree possible. I'm not going to use the word to know. Oh, to measure. You're going to use the word to measure. Good. Measure is a very slippery word, too. I've used it here because I couldn't really get away with not using it. But we'll talk about that in some detail later on in the course. For the moment, I want to emphasize that it's tempting but dangerous at this point to talk about whether you know or don't know or whether someone knows or doesn't know. For example, the monkey inside knows or doesn't know, right? So let's try to avoid that and focus on just operational questions of what are the things that go in, what are the things that come out, And the reason that's so useful is that it's something that you can just do. There's no ambiguity about whether you've caught a white electron in a particular spot. Now, in particular, the reason these boxes are such a powerful tool is that you don't measure the electron, you measure the position of the electron. You get hit by the electron or you don't. And by using these boxes, we can infer from their position the color or the hardness. And that's the reason these boxes are so useful. So we're inferring from the position, which is easy to measure if you get beamed or you don't. We're inferring the property that we're interested in. So it's a really good question, though. Keep it in the back of your mind, and we'll talk about it on and off for the rest of the semester. Yeah? So what happens if you have a setup and you just take away the bottom right here? Perfect question. This leads me into the next experiment. So here's the modification that, thank you, that's a great question. Here's a modification of this experiment. So let's rig up a small, hold on, I want to go through the next series of experiments and then I'll come back to questions. And these are great questions. So I want to rig up a small movable wall, a small movable barrier. And here's what this movable barrier will do. If I put the barrier in, so this would be in the soft path, when I put the barrier in the soft path, it absorbs all electrons incident upon it and impedes them from proceeding. So you put a barrier in here, put a barrier in the soft path, no electrons continue through it. An electron incident cannot continue through it. When I say that the barrier is out, what I mean is it's not in the way. I've moved it out of the way. Cool? So I want to run the same experiment. And I want to run this experiment using the barriers to tease out how the electrons transit through our apparatus. So experiment four, let's send in a white electron again. I want to do the same experiment we just did and color it out, but now with a wall in the south path. Wall in soft. So that's this experiment. So we send in white electrons. And we send out, and at the output we measure the color as before. And the question is, what fraction come out black and what fraction come out white? OK? So again, everyone think through it for a second. Just take a second. And this one's a little sneaky, so just feel free to discuss it with the person sitting next to you. All right. All right, now that everyone's had a quick second to think through this one, let me just talk through what I'd expect from the point of these experiments, and then we'll talk about whether this is reasonable. So the first thing I expect is that, look, if I send in a white electron, I put it in a hardness pass, I know that 50% of the time it goes out hard and 50% of the time it goes out soft. If it goes out the soft aperture, it's going to get eaten by the barrier, right? It's going to get eaten by the barrier. So first thing I predict is that the output should be down by 50%. However, here's an important bit of physics, and this comes to the idea of locality. I didn't tell you this, but these arm lengths in the experiment I did, 3,000 kilometers long. 3,000 kilometers. That's too minor. 10 million kilometers long. Really long. Very long. Now, imagine an electron that enters this, an initially white electron. If we had the barriers out, if the barrier was out, what do we get? 100% white, right? We just did this experiment to our surprise. So if we did this, we get 100%. That means that any electron going along the south path comes out white. Any electron going along the heart path goes out white. They all come out white. So now, imagine I do this. Imagine we put a barrier in here, 2 million miles away from this path. How does a heart electron along this path know that I put the barrier there? And I'm going to make it even more sneaky for you. I'm going to insert the barrier along the path after I launch the electron into the apparatus. And when I send in the electron, I will not know at that moment, nor will the electron know, whether, because they're not very smart, whether the barrier is in place. And this is going to be millions of miles away from this guy. So an electron out here can't know. It hasn't been there. It just hasn't been there. It can't know. But we know that when we ran this apparatus without the barrier in there, it came out 100% white. But it can't possibly know whether the barrier is in there or not. It's over here. So what this tells us is that we should expect the output to be down by 50%. But all the electrons that do make it through must come out white, because they didn't know that there was a barrier there. They didn't go along that path. Yeah. Oh, I was just sorry. No, thank you. Thank you. Thank you. That was a slip of the tongue. I was making fun of the electron. So in that particular case, I was not referring to my or your knowledge. I was referring to the electron's tragically impoverished knowledge. Yeah. Well, here's the more troubling thing. Imagine it didn't come out 100% white. Then the electron would have demonstrably not gone along the soft path. It would have demonstrably gone through the hard path, because that's the only path available to it. And yet, it would still have known that millions of miles away, there's a barrier on a path it didn't take. So which one's more upsetting to you? And personally, I find this one the less upsetting of the two. So the prediction is our output should be down by 50%, because half of them get eaten. But they should all come out white, because those that didn't get eaten can't possibly know that there was a barrier here, millions of miles away. OK. So we run this experiment. And here's the experimental result. In fact, the experimental result is, yes, the output is down by 50%. But no, not 100% white, 50% white. 50% white. So the barrier, if we put the barrier in the hardness path, if we put the barrier in the hardness path, still down by 50%. And it's at odds, 50-50. How could the electron know? I'm making fun of it. Yeah. So I guess my question is, before we ask how it knows there's a plot for one of the paths, how did it know before over there that there were two paths? Excellent. Exactly. So actually, this problem was there already in the experiment we did. All we've done here is tease out something that was existing in the experiment, something that was disturbing. The presence of those mirrors and the option of taking two paths somehow changed the way the electron behaved. How is that possible? And here, we're seeing that very sharply. Thank you for that excellent observation. Yeah. What if you replaced the two mirrors with color boxes so that both color boxes would have their hardness option, and you would recombine all four beams and then put them in the last color box? Yeah, so that's a totally reasonable. So the question is basically, let's take this experiment and let's make it even more intricate by, for example, replacing these mirrors by color boxes. OK, so here's the thing I want to emphasize. Something that, well, how do I say it? I strongly encourage you to think through that example. And in particular, think through that example. Come to my office hours and ask me about it. So that's going to be studying a different experiment, and different experiments are going to have different results. So we're going to have to deal with that on a case by case basis. It's an interesting example, but it's going to take us a bit afar from where we are right now. But after we get to the punch line from this, come to my office hours and ask me exactly that question. OK, yeah. So we have a color box, we put it in white, there's like electrons, and we're going to take the distance, like random stuff. Yeah. How do you know the boxes work? How do I know the boxes work? These are the same boxes we used from the beginning. We tested them over and over. How did you first check it if it worked, if it's always trying to make a difference? How to say? There's no other way to build a box that does the properties that we want, which is that you send in color and it comes out color again, and the mirrors behave this way. Any box that does those first set of things, which is what I will call a color box, does this too. There's no other way to do it. I mean, I don't mean this because, like, you know, no one's, you know, Oh, sure, you can. You take the electron that came out of the color box. That's what we mean by saying it's white. Well, but that's it. But that's what it means to say the electron is white, right? It's like, how do you know that my name is Alan? You say, Alan, I go, what? Right. But you might feel like, look, that's not a test of whether I'm Alan. It's like, well, what is the test of what you that's how you test? What's your name? I'm Alan. Oh, great. That's your name. So that's what I mean by white. Now, you might quibble that that's a stupid thing to call an electron. And I grant you that. But it is nonetheless a property that I can empirically engage. OK, I was I've been told that I never ask questions from the people on the right. Yeah. No, this experiment has been done against again by some French guys. There's this the French. Look, dude. So there's this guy, Alana Spey. Oh, great experimentalist, great physicist. And he's done lots of beautiful experiments on exactly this topic. And send me an email. And I'll post some some example papers and reviews by him. And he's a great writer on the Web page. So just send me an email to remind me that. OK, so we're lowish on time. So let me move on. So what I want to do now is I want to take the lesson of this experiment and the observation that was made a minute ago that, in fact, the same problem was present when we ran this experiment and got 100 percent. We should have been freaked out already. And I want to think through what that's telling us about the electron, the single electron as it transits the apparatus. The thing is, at this point, we're in real trouble. And here's the reason. Consider a single electron inside the apparatus. And I want to think about the electron inside the apparatus while all walls are out. So it's this experiment. Consider the single electron. We know with total confidence, with complete reliability, that every electron will exit this color box out the white aperture. We've done this experiment. We know it will come out white. Yes? Here's my question. Which word did it take? Not a spoiler. Which route did it take? I'm asking you the question. That's why you care. I'm the professor here. What is this? Come on. Which route did it take? OK, let's think through the possibilities. Grapple with this question in your belly. Let's think through the possibilities. First off, did it take the hardness path? So as it transits through the single electron transiting through this apparatus, did it take the hard path? Or did it take the soft? These are millions of miles long, millions of miles apart. This is not a ridiculous question. Did it go millions of miles that direction, millions of miles that direction? Did it take the hardness path? Ladies and gentlemen, did it take the hardness path, the hard path? Well, we ran this experiment by putting a wall in the soft path. And if we put a wall in the soft path, then we know it took the hard path because no other electrons come out except those that went through the hard path, correct? On the other hand, if it went through the hard path, it would come out 50% of the time white, 50% of the time black. But in fact, in this apparatus, it comes out always 100% white. It cannot have taken the hard path. No. Did it take the soft path? Same argument, different side. Right? No. Well, this is not looking good. Well, look, maybe there was, there was a, this, this was suggested. Maybe it took both. Maybe electrons are sneaky little devils that split and do, and part of it goes one way, and part of it goes the other. Maybe it took both paths. So this is easy. We can test this one. And here's, I'm going to test this one. Uh. Oh, sorry. Actually, I'm not going to do that yet. So we can test this one. So if it took both paths, here's what you should be able to do. You should be able to put a detector along each path. And you'd be able to get half an electron on one side and half an electron on the other, or maybe two electrons, one on each side and one on the other, right? So this is the thing that you'd predict if you said it went both. So here's what we'll do. We will take detectors. We will put one along the hard path and one along the soft path. We will run the experiment and then observe whether and ask whether we see two electrons, we see half and half, what do we see? And the answer is you always, always see one electron on one of the paths. You never see half an electron. You never see a squishy electron. You see one electron on one path, period. It did not take both. You never see an electron split in two, divided, confused. No. Well, it didn't take the hard path, didn't take the soft path. It didn't take both. There's one option left. Neither. Well, I say neither. But so what about neither? And that's easy. Let's put a barrier in both paths. And then what happens? Nothing comes out, right? So no. So now, to repeat an earlier prescient remark from one of the students, what the hell? So here's the world we're facing. I want you to think about this. Take this seriously. Here's the world we're facing. And when I say here's the world we're facing, I don't mean just these experiments. I mean the world around you, 20 kilo mirrors, buckyballs. Here is what they do. When you send them through an apparatus like this, every single object that goes through this apparatus does not take the hard path. It does not take the soft path. It doesn't take both. And it does not take neither. And that pretty much exhausts the set of logical possibilities. So what are electrons doing when they're inside the apparatus? How do you describe that electron inside the apparatus? You can't say it's on one path. You can't say it's on the other. It's not on both. And it's not on either. What is it doing halfway through this experiment? So if our experiments are accurate to the best of our ability to determine they are, and if our arguments are correct, and that's on me, then they're doing something. These electrons are doing something we've just never thought of before, something we've never dreamt of before, something for which we don't really have good words in the English language. Apparently, empirically, electrons have a way of moving. Electrons have a way of being, which is unlike anything that we're used to thinking about. And so do molecules. And so do bacteria. So does chalk. It's just harder to detect in those objects. So physicists have a name for this new mode of being. And we call it superposition. Superposition. Now, at the moment, superposition is code for, I have no idea what's going on. A usage of the word superposition would go something like this. An initially white electron inside this apparatus, with the walls out, is neither hard nor soft, nor both, nor neither. It is, in fact, in a superposition of being hard and of being soft. And this is why we can't meaningfully say this electron is some color and some hardness. Not because our boxes are crude and not because we're ignorant, though our boxes are crude and we are ignorant. It's deeper. Having a definite color means not having a definite hardness, but rather being in a superposition of being hard and being soft. Every electron exists, or sorry, every electron exits a hardness box, either hard or soft. But not every electron is hard or soft. It can also be a superposition of being hard or being soft. The probability that we subsequently measure it to be hard or soft depends on precisely what superposition it is. For example, we know that if an electron is in the superposition corresponding to being white, then there are even odds of it being subsequently measured to be hard or to be soft. So to build a better definition of superposition than I have no idea what's going on is going to require a new language, and that language is quantum mechanics. And the underpinnings of this language are the topic of the course, and developing a better understanding of this idea of superposition is what you have to do over the next three months. Now, if all this troubles your intuition, well, it shouldn't be too surprising. Your intuition was developed by throwing spears and, you know, running from tigers and catching toast as it jumps out of the toaster, right? All of which involves things so big and with so much energy that quantum effects are negligible. As a friend of mine likes to say, you don't need to know quantum mechanics to make chicken soup. However, when we work in very different regimes, when we work with atoms, when we work with molecules, when we work in the regime of very low energies and very small objects, your intuition is just not a reasonable guide. It's not that the electrons, and I cannot emphasize this strongly enough, it is not that the electrons are weird. The electrons do what electrons do. This is what they do. And it violates your intuition, but it's true. The thing that's surprising is that lots of electrons behave like this. Lots of electrons behave like cheese and chalk. And that's the goal of 8.04, to step beyond your daily experience and your familiar intuition and to develop an intuition for this idea of superposition. And we'll start in the next lecture. I'll see you on Thursday. Thank you.
    

## 3. (improved) Create a Summary with ChatGPT


I will invoke the following strategy. 

1. Reformat the transcription to reduce token usage but keep timestamp information.
2. Break the video up into approximately 10 separate chunks.
3. Ask ChatGPT to summarize each chunk into valuable notes.
4. Send ChatGPT it's collective summarize and ask it to combine them.

Step 3 gives increased granularity and visibility into our process of breaking up our information. 
The model missing two important concepts in our prompts (locality and uncertainty principle) could be due to the fact that they're simply not mentioned as much 
as the terms **superposition** or **quantum mechanics**.
This appears necessary given the model's inconsistency in dealing with all of the text at one time. 
We can also break this up further to have some sort of note taken transcript that more closely follows how we would take notes on our own.
In general, grabbing a full summary of a lecture is how we end up remembering the information in our minds.
However, the visual portion of this video is still important (equations, graphs, demonstrations, etc.) to solidify those memories more easily.
This won't be fixed in this video but will be addressed later in the article's conclusion.



```python

# Load the transcription segments from the file

transcription_file = "..\data\output_transcription.json"

with open(transcription_file, 'r') as file:
    transcription_segments = json.load(file)

```


```python
from pprint import pprint 

time_stamped_transcription = ""
for segment in transcription_segments:
    hour = int(segment['start']//3600)
    minute = int((segment['start']%3600)//60)
    seconds = int(segment['start']%60)
    time_stamped_transcription += f"[{hour}:{minute}:{seconds}] {segment['text']} \n"
    
pprint(time_stamped_transcription[0:1000])
```

    ('[0:0:0]  The following content is provided under a Creative \n'
     '[0:0:2]  Commons license. \n'
     '[0:0:3]  Your support will help MIT OpenCourseWare \n'
     '[0:0:6]  continue to offer high quality educational resources for free. \n'
     '[0:0:10]  To make a donation or to view additional materials \n'
     '[0:0:12]  from hundreds of MIT courses, visit MIT OpenCourseWare \n'
     '[0:0:16]  at ocw.mit.edu. \n'
     '[0:0:21]  Hi, everyone. \n'
     '[0:0:24]  Welcome to 8.04 for spring 2013. \n'
     '[0:0:27]  This is the fourth and presumably final time \n'
     "[0:0:30]  that I will be teaching this class, so I'm pretty excited about "
     'it. \n'
     '[0:0:33]  So my name is Alan Adams. \n'
     "[0:0:34]  I'll be lecturing the course. \n"
     "[0:0:37]  I'm an assistant professor in course eight. \n"
     '[0:0:40]  I study string theory and its applications \n'
     '[0:0:43]  to gravity, quantum gravity, and condensed matter physics. \n'
     '[0:0:48]  Quantum mechanics, this is a course in quantum mechanics. \n'
     '[0:0:52]  Quantum mechanics is my daily language. \n'
     '[0:0:54]  Quantum mechanics is my old friend. \n'
     '[0:0')
    


```python
import math

llm = ChatOpenAI(model_name="gpt-3.5-turbo")

# Calculate the number of segments in each 5 minute group
num_segments = len(transcription.segments)
segments_per_group = math.ceil(num_segments/20)

# Create a list to store the grouped segments
segment_groups = []

# Iterate over the segments and group them
for i in range(0, num_segments, segments_per_group):
    group = transcription.segments[i:i+segments_per_group]
    segment_groups.append(group)
```


```python

output_parser = StrOutputParser()
prompt = ChatPromptTemplate.from_messages([
    ("system", """
     You are world class note taker and will summarize the following transcript from a lecture. 
     Summarize into as many key points as possible and definitions.
     Format it as such.
     Key points:
     [key points]
     
     Definitions:
     [definitions]
     """),
    ("user", "{input}")
])

chain = prompt | llm | output_parser


output_summaries = []
for group in segment_groups:
    output = chain.invoke({"input": group})
    output_summaries.append(output)

# print only the last output
print(output)

    
```

    Key points:
    - Physicists have a term for a new mode of being called "superposition."
    - Superposition refers to an object being in a state of two or more contradictory properties simultaneously.
    - Electrons can exist in a superposition of being hard and soft.
    - Quantum mechanics provides a language to understand superposition in the realm of atoms and molecules.
    - Developing an intuition for superposition is essential in quantum mechanics.
    - Electrons exhibit behavior in superposition, which may defy traditional intuition.
    - The course aims to expand understanding beyond daily experiences to grasp the concept of superposition.
    
    Definitions:
    - Superposition: The state in which an object simultaneously exists in multiple contradictory states or properties.
    - Quantum mechanics: The branch of physics that deals with the behavior of particles at the atomic and subatomic levels.
    

Now I'll compile all of these notes into a giant query and send it for a meta-summarization.


```python
# combine the text summaries into a single string
text_summary = ""
for summary in output_summaries:
    text_summary += summary + "\n\n"

    
# create a meta chain
meta_prompt = ChatPromptTemplate.from_messages([
    ("system", """
    Compile and organize the following notes together into two categories: key points and definitions.
     """),
    ("user", "{input}")
])


meta_chain = meta_prompt | llm | output_parser

note_output = meta_chain.invoke({"input": text_summary})
print(note_output)
```

    ### Key Points:
    - The lecture focuses on teaching quantum mechanics with an emphasis on developing intuition rather than just calculations.
    - Active participation, problem-solving, and collaboration with other students are encouraged to enhance understanding.
    - Course materials are available on the MIT OpenCourseWare website.
    - Clickers are used for non-graded activities like concept questions and quizzes.
    - Two midterms and one final exam are part of the assessment.
    - Recommended textbooks are suggested for further reading.
    - Asking questions, attending office hours, and recitations are advised for better understanding.
    - The lowest problem set score will be dropped without questions.
    - Importance of group work to efficiently cover all recommended textbooks.
    - The unpredictability and randomness in quantum mechanics are highlighted.
    - The limitations of building specific boxes due to fundamental principles are discussed.
    - The uncertainty principle in physics is introduced.
    - Experiments are conducted to observe the behavior of electrons and photons.
    - The concept of superposition in quantum mechanics is explained.
    
    ### Definitions:
    - **Quantum mechanics:** A branch of physics that deals with the behavior of particles at the atomic and subatomic levels.
    - **MIT OpenCourseWare:** An initiative by MIT to provide free access to course materials from MIT classes.
    - **Problem sets:** Assignments given to students to solve and practice the concepts taught in class.
    - **Clickers:** Electronic devices used for interactive participation and feedback in lectures.
    - **Midterms:** Examinations held in the middle of the semester to assess students' understanding of the course material.
    - **Final exam:** Comprehensive exam held at the end of the semester to evaluate students' overall knowledge of the course.
    - **Collaboration:** Working together with other students to achieve common goals.
    - **Textbooks:** Academic books containing information and explanations on a particular subject.
    - **Office hours:** Designated times outside of class where students can meet with the professor for additional help or clarification.
    - **Recitations:** Small group sessions led by teaching assistants to review course material and work on problem-solving.
    - **Lecture series:** A series of lectures on a particular topic or subject matter.
    - **Reading assignments:** Required readings for a course to supplement the material covered in lectures.
    - **Color boxes:** Containers where electrons are sent in and may come out either white or black.
    - **Hardness boxes:** Containers where electrons are sent in and may come out either soft or hard.
    - **Correlation:** A relationship or connection between two or more variables, indicating they may influence each other.
    - **Probability:** The likelihood of a specific outcome occurring in an event or experiment.
    - **Determinism:** The philosophical concept that all events are determined by external causes.
    - **Compression ratio:** The level of compression applied to speech data.
    - **No speech probability:** The likelihood of non-speech elements in the audio.
    - **Aperture:** An opening or hole through which particles can pass.
    - **Photons:** Particles of light that carry electromagnetic force.
    - **Laser:** Device that emits light through optical amplification.
    - **Apparatus:** Equipment or tools used for a specific purpose.
    - **Empirical:** Based on observation or experience rather than theory.
    - **Superposition:** The state in which an object simultaneously exists in multiple contradictory states or properties.
    
    By compiling and categorizing the content, the key points and definitions are presented in a structured and organized manner for better understanding.
    

This is an improvement in the shear amount of information created.
And although we did pick up definitions on the **uncertainty principle** (found in key concepts not definitions) and **determinism**,
we did not find **locality**.
Again, this can come down to prompt engineering or how well we break up the incoming information.
From this point of view I have a "good enough" solution that at least mentions 80% of the new class specific definitions (locality is mentioned once with a vague definition, it could come up in future lectures with more emphasis).
The video is still important to watch, but if we wanted to follow the class and say, watch a independent videos on those specific definitions, this is a great way to compile a list of where to start.


# Conclusion

I generally try to produce projects that I would use in the future, but the amount of work required to make this particularly useful to me requires more experience in other domains and lot more time.
A good application to this process would be some sort of web plugin or video player plugin, that pre-generates the notes and gives us real time contextual basis and maybe definitions as we go.

If you pay attention closely to the original lecture, many of the definitions provided by ChatGPT in our last version of the project were not explicitly stated by the lecturer.
There are pros and cons to this approach. On one hand, we get very basic definitions of some definitions as trivial as "trivial". 
This is great because not everyone really understands what that means.
This is also bad because part of the learning process is developing a vocabulary for the subject matter by context clues of the surround statements.
Too much learning can be taken away from the student.
Another con is, how can we be absolutely certain that in-lecture definitions ARE the definitions defined within the lecture?
ChatGPT may be using it's own definition.
For simple subjects, this is probably ok.
But for more complex topics, having an expert explain it to you in it's own words is important.

This brings me to my ideal use case for this type of note summarizing with large language models. 
It would be either an application itself, or a plugin to a popular video player such as VLC.
The application would alter the lecture process as such.

1. A video player that contains some sort of interaction box below it for students to follow along with.
2. When something not basic is said, such as a definition that is not immediately defined, it will appear either with the definitions or for the student to click on.
3. When information that is relevant to the class pops up, you may save that information as class relevant, to appear at any time.
   1. The LLM will help provide those definitions, both with what is defined by the professor, and the most common definition.
4. Periodically throughout the video, your definition and problem solving ability will be testing with small quizzes.
   1. This would be a sub application of the LLM by having it identify key information that needs remembered, and asking you to define it sometime later in the lecture after you've already learned the definition.
5. For real time classes, relevant information such as test and quiz dates could be parsed and handled.
6. Going even further, maybe there is some sort of account management where you can share notes with other students, corrections to syllabus details, etc.

Lectures, in my eyes, are one of the lowest forms of learning. They require us to be active listeners to gain anything of value, but do not punish us for simply tagging along.
Altering the lecture watching process with periodic check-ins and real-time interactions would make it easier to stay engaged and pay attention.
All these little moments of learning add up and can prevent students from cramming, which is well known for producing high amounts of stress, and poor short/long term results.

