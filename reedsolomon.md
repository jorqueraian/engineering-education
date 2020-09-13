
# Reed-Solomon Error Correction

Have you ever wondered how DVDs can still be played after they have been scratched? This is done with Reed-Solomon error-correcting codes. This might not seem, relevant to our internet of things world but the ideas behind it are still very important. Reed-Solomon codes are prominently used in many forms of broadcasting and in storage systems that use RAID 6. 

Before reading this article I would recommend reading my introduction to error correcting codes article [parts 1](https://www.section.io/engineering-education/understanding-error-correcting-codes-part-1/) and [2](https://www.section.io/engineering-education/understanding-error-correcting-codes-part-2/). They provide intuition behind the definition and creation of simple error-correcting codes.

Unlike more traditional error-correcting codes, like the Hamming code, that correct bits, Reed-Solomon can correct bytes. This gives the code the ability to correct large clusters of error, as it doesn't matter how bits in each byte experience get flipped. This clustering effect allows for vast segments of contiguous data to be correctable that wouldn't normally be fixed by traditional codes.
  

This article will focus on the high-level basic intuition behind Reed-Solomon Codes. For a mathematical understanding, I recommend looking over the lecture notes in the sources section. Alternatively, an implementation of the Reed-Solomon code in python can be found [here](https://repl.it/@jorqueraian/ReedSolomon).

  

## How do they work?

Error-correcting codes work with what are called finite fields. A [field](https://mathworld.wolfram.com/Field.html) is a set of elements, or numbers, that have the operations addition, subtraction, multiplication, and division well defined. Fields must also satisfy a list of [axioms](https://mathworld.wolfram.com/FieldAxioms.html).  An example of a field and specifically an infinitely large field is the set of a real number $\mathbb{R}$ with the normals operations for addition subtraction, multiplication, and division. A finite field on the other hand is a field with a finite number of elements or numbers and an example of this is the set $\{0, 1\}$. For this field, multiplication and division will behave as you expect, and addition and subtraction are both defined with the XOR operator. Notice that with this set no operation will result in a value other than 0, or 1. This is called closure and must be true for all fields, for each of the operations. 

Both the Hamming and the Golay codes as presented previously uses finite fields of just 2 elements, 0 and 1. Because the Reed-Solomon code can correct entire bytes it uses a finite field of 256 elements, one element representing each possible value of a byte. We can represent these values with the integers from 0 to 255. Addition and subtraction will be defined with the XOR operator. For example consider the numbers 26 and 14, that correspond to `00011010` and `00001110` respectively. The XOR of these numbers is then `00010100` which is the number $20$. So $26+14 = 26-14=20$. In this finite field of 256 elements, multiplication is polynomial multiplication where it treats each number as a polynomial with the bits as coefficients. To see how this field is used check out this interactive python implementation [here](https://repl.it/@jorqueraian/FiniteFields). 
  

As mentioned previously using this field of 256 elements allows us to formulate error-correcting codes that can correct entire bytes instead of bits. With the Reed-Solomon, an input message is a grouping, our a vector, of bytes rather than bits. For example if our input was "Hello World" it would be the vector $[72, 101, 108, 108, 111, 32, 87, 111, 114, 108, 100]$. 

The Reed-Solomon code can be implemented to work with any desired input size and encoded message size, meaning we can add however many redundant bytes we desire. The more redundant bytes we add the more correctable bytes there are. We can create a code that will encode messages of size $k$ to encoded messages of size $n$. This means if we set $k=24$ $n=30$ we will take in messages of 24 bytes, add 6 redundancy bytes to create the encoded message of 30 bytes. This will give us the ability to correct three bytes while having 80% of the encoded message being made up of the original data. In general, the Reed-Solomon code allows us to correct $\lfloor(n-k)/2\rfloor$ bytes for any $k$ and $n$.

  

The Reed-Solomon code works by creating a list of $n$ code locators, in our case 30 code locators, one for each element or byte in the encoded message. Each code locator will be unique and correspond to a specific position in the encoded message. For example, the first code locator will correspond to the first element in the encoded message, and similarly, the 18th code locator will correspond to the element in the 18th spot or the 18th byte of the encoded message. We encode our message with these code locators so when and if an error occurs we can determine its location depending on which code locator it affects.

### The Encoding Process

To encode a message with Reed-Solomon we create a polynomial from the input message, with each of the bytes being a coefficient. So with our 24-byte input message $m= m_0,\dots,m_{23}$ we will create the function $m(x) = m_0 + m_1x+m_2x^2+\dots+m_{23}x^{23}$. For example, if the first 3 bytes of our message were $(200, 123, 3)$, the first three terms of the polynomial would be $200 + 123x + 3x^2$. To them create the actual encoded message we plug in each of the code locators into this function and once we have run this function for each of the 30 code locators we will have our 30-byte encoded message.

  

### The Decoding Algorithm

  

As brought up in the previous article linear error-correcting codes use two matrices, one to encode and the second to calculate the syndrome of the received messaged, which is used to determine what errors occurred and where they are. Reed-Solomon is a linear code so these two matrices can be created from the polynomial used to encode the message, presented in the previous section. The generator matrix will have columns corresponding to the code locators. To see what these matrices will look like, check out the lecture notes in the sources section. In general, using the polynomial to encode the message will be faster, but both the polynomial and the matrix will result in an identical encoding.

There are a variety of Reed-Solomon decoding algorithms one of which is called the Peterson-Gorenstein-Zierler Generalized Reed-Solomon decoding algorithm. This algorithm creates a polynomial called the error locator polynomial, from the code locators and the syndrome of the received message. The roots of this polynomial correspond to the locations of the errors that occurred, represented by the code locators. So once we have this polynomial we can plug in each of the code locators, or more accurately the multiplicative inverses of the code locators, to determine which positions in the encoded message had an error. To then correct these erroneous bytes we can create a solvable system of equations that when solved gives us the errors, for each of the erroneous bytes.
  

This process of decoding is by no means "fast" as solving systems of linear equations is not an easy problem. In real-world situations, such as DVD players this process uses specialized hardware allowing these calculations to be performed in real-time.

 
To see an interactive implementation of both the encoding and decoding algorithms for text inputs and randomized errors look [here](https://repl.it/@jorqueraian/ReedSolomon).

  

  

## Visualizing the Reed-Solome code

  

Now that we have a basic understanding of Reed-Solomon codes let's see how they perform. In this example, I will use a Reed-Solomon code that takes in messages of 64 bytes and encodes them into messages of 82 bytes. This will make it possible for the code to correct up to 9 bytes of errors for each 82-byte message. I will use an image of Jupiter for this example where each 64-byte message is roughly 21 pixels. This message will then be encoded into an 82-byte encoded message.

  

<p align="middle"><img width="450" height="450" src="https://lh3.googleusercontent.com/JPCtdgvtAMqRPTGM0zuZAdvCiHAePwOXciB_JVPSmROw6jSkCWl-7zg3aeCn2Ra_ORqJYSwVk9KMTj2Q8LjwD05fm-wy4rNDIncnx13oRXEHD6kqCoZDLI39mT8k3D4f2-Ks6SP9ZKO9mI6wRaHxG5A0F4OS1HLODYhH5_7Xqf6xcay7GF8rsK_CDiTm2xDmJe23-Mn4BJlgPVAg3hJvyfru68iKtt7W4rsD67pL-6BRCzVe8OnEvX0K8D_0MYZjUiAm8v4Tavv6SJvYHJnORBKRAwiGjoyyT473joOJsopIRCFQZMYvejU7zxjdHbe1J7gH3CvZ9B9dWcconyITvhLDkN8vQxgYukHJlTM9rkaw9h3SchV-de7zYs2QINTGU2Ltg7DBy1TRClmwlf-oa6hJ-RM-s35r57ermVxQ2dck2D-ZBTAW39SbwGfYT7Hnj8FpB45V2STGHyg814a21x5QWXQ0IIQ857wLmBS_j8hbwHWTVpq6qH9xKUd5lV9s1zqCzg8WGjMo-ULDHTGbT9fIe4cuwTJWZ0SfcLiZAAus9uL3i9TizesOxOaNelgg9Xf6brKYpV5vbpmVPsQtzOgWKY2UdLNko8RXZE5Nzl5T1hhH9VxstO8PeWPYIdO3St3Xka2T2RhVgk9oGHie_iaMnvh8RDd5Q-oQwzUBuO_peui9_ry-0ES-KRMAoF1KFVWQISruZc2acjkoilXdG_mhAMs2E_IvcgaFyxo7LmeL5hlFpS-wEkQ=s640-no"/></p>

  

Once the image was encoded I simulated a scratch. Each white pixel in the scratched image represents 3 consecutive bytes or error, one byte for each color value. This corresponds to 72 bits or 3 pixels of consecutive error, which would not be possible for either the Hamming or the Golay code to correct. 

  

<p align="middle"><img width="450" height="450" src="https://lh3.googleusercontent.com/V4xEn_lZz4EQo1_Fv6UINK87CaOioyLJrZ6U_8S-Jzf_v9KO_DbO4s5m4az9F3YcDtFbd6Y3P9W_9M3zeJc0O5C5Vn7z9FMRQRpFfzEaA0Gs3HvoH13PkoArIc9e1BCdCMSsemjdBimIR0lqvchTnv-jmpJM7Qw3JpZ_btSbxEYQoH2xf1jdG0xCw5YAby7O6aUXL-m_qr7BQLntPjozmH5m734Ko5mvhKIpk0eO-EMEdb2-K9ebw9j8pEbffL1-3glefcAwfWPIoChAq3ledMmw4iC3lm1JavclFvix5sIZsLRuU4NfwsjGowDXtb0FeI_WrjeCJ4-Hiq-5yAypdRlSk6l1ScmDNlx5rZq3d_4k-S0kXsRQLoGp6fTdZRhGMaSlhYxavMFlxozeRW1ZqJpp-f35nnEHoa2cSFpHtc1uA-_SSJpHB8qAn7jm-fw5HM3M1AWkbVmm-M9MVUYQ1BOxDVFFfPUASmxzy0-Nz9Ovc8udqGBut43jmQYVTXIcfcabcHIzvpRrzQfUXGNrS4B-7Fy4xyzAc7g9-JedeaIurt1xeMVX8_oZNxSeUnxJsNONsFGKqNkkITTh76E-m_DQ14-u0Tjf_5JJPGVG3JkczVgZWNNeY46bvrcvcVXWmxuI3ORvsUeEzAKvDSVfFHjTpXIwjZ1D3Aor1gZaNe9PZziK5k2fNm2lzGo_ZdYna2l1XTcHzpCRaua1kXzdWJ3deRhU2X9GzDdkn6W5HaQM4Tnpk0I8taQ=s640-no"/></p>

Now we can decode the image. As we see, most of the scratch was accurately decoded. There are only two visible instances of error left which can be seen as black bars in the image. This is very good especially considering the "scratch" had an average width of about 2 pixels or 6 bytes.

<p align="center"> <img width="450" height="450" src="https://lh3.googleusercontent.com/7HfAlqgNozeFmaoUKCincy9m2jpl_hiEjhWgcGp1tZ39hDDlQpawpuycuj-LT-yIyCVWa-KdHExtdyYoeHfB3z1MrRDci8u31XCkx-uGEeca_mIEZmJKxp9HYOhg51FFxLl0qXK8KExhhzFdwqzLfo_UX6-3n8IWehCZUHMaWTeh74bX2ci6HGhOB1TaNoDd2I5-sPx57HOiTOhgbGao-LKYefq6pirLTbe-bpgi00ivmQHDZHPwTtvODsYE6m93NqApQoL-kWsQGLkf4f6Jw6-iBjvqujZsWpCxjqOqELd77ucGUlOQWYZB5ZAidtWu4Lz1MarzAGx2cIRnUbIBNW7B1nwsBnqLqtawCRBx9Lcbt_upqknTyNiI9lKfl2G3kciWCglUED-gHAnH03VG1L5g4YOvO4SNg1N4ZesXHPjVr9xzJ_Qdh0mM_XUN0blEhEaWtLzrWhZYdN2ybqKHoa3BhuFAm_UzleMtPhpcX2cLLWRQEFXA0ED7ZeQGTF4P5vQmm72GITEvSrG5uohRPfebRJZCPiqWwxKTyrcW-jM-ZCY2JrlGROZPo2qlgsqYOctxdE4lp2eYSA8mRc3uyLx4GZJwiN8UhUVTdRN-gNHJTyVqwt1ab25p9SEVgusqi4hmuyfX4_qLacoidiJnSeWyR1e7OFFr_kOnaHjVRrzKglaKem_VOqR1crtlcSUS8UFNyOOX2l_V4_-5Fr4XogonQp6p8j3dgjMdR3xE1MmyUDUaOthOcmk=s640-no"> </p>

  

Additionally, the error is sparse meaning the pixel values of these areas could be approximated from the average of the surroundings to give an even better-corrected image. This is exactly what happens to errors too large to correct on DVDs.

  

## Further reading

If you're interested in learning more about error-correcting codes, specifically with a focus on the mathematics behind them, I would suggest reading the lecture notes, [here](http://u.cs.biu.ac.il/~lindell/89-662/main-89-662.html).

Some other interesting error-correcting are the [Reed-Muller code](https://en.wikipedia.org/wiki/Reed%E2%80%93Muller_code), [Polar codes](https://en.wikipedia.org/wiki/Polar_code_(coding_theory)), and the different types of [Concatenated codes](https://en.wikipedia.org/wiki/Concatenated_error_correction_code), such as the Forney code, all of which are mentioned in the lecture notes provided in the Sources section.

## Sources

Lindell, Y. Introduction to Coding Theory (89-662) [Lecture Notes]. (2010). Retrieved from (http://u.cs.biu.ac.il/~lindell/89-662/main-89-662.html)