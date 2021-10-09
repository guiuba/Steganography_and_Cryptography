# Steganography_and_Cryptography
This program asks for a message and a password. Then encrypts and conceals the message in an image.  It shows back the message, if the password is correct : )
<import java.awt.Color
import java.awt.image.BufferedImage
import java.io.File
import javax.imageio.IIOException
import javax.imageio.ImageIO
import kotlin.experimental.xor

var exit = false
fun main() {
    while (!exit) {
        showMenu()
    }
}

fun showMenu() {
    println("Task (hide, show, exit):")
    when (val input = readLine()!!) {
        "hide" -> hideMessageInImage()
        "show" -> showMessageInImage()
        "exit" -> {
            exit = true
            print("Bye!")
        }
        else -> println("Wrong task: $input")
    }
}

fun hideMessageInImage() {
    println("Input image file:")
    val inputFileName = readLine()!!
    println("Output image file:")
    val outputFileName = readLine()!!
    println("Message to hide:")
    val message = readLine()!!
    println("Password:")
    val password = readLine()!!
    try {

        val inputFile = File(inputFileName)  //eg."winter.png"
        val image: BufferedImage = ImageIO.read(inputFile)
        val imageSizeBlue = image.height * image.width
        val messageSize = message.length * 8 + 24
        if (messageSize > imageSizeBlue) {
            println("The input image is not large enough to hold this message.")
            showMenu()
        }

        val outputImage = hideTextInImage(image, encryptMessage(message, password))
        val outputFile = File(outputFileName)
        ImageIO.write(outputImage, "png", outputFile)
        println("Message saved in $outputFileName image.")
    } catch (e: IIOException) {
        println("Can't read input file!")
        showMenu()
    } catch (e: IllegalArgumentException) {
        println("parameter is null!")
        showMenu()
    }
}

fun encDecXorFun(message: ByteArray, password: ByteArray): ByteArray {
    val encryptedMessageByteArray = ByteArray(message.size)
    var counter = 0
    for (i in message.indices) {
        if (i > password.size - 1) {
            encryptedMessageByteArray[i] = message[i] xor password[counter]
            counter++
            if (counter > password.size - 1) {
                counter = 0
            }
        } else
            encryptedMessageByteArray[i] = message[i] xor password[i]
    }
    return encryptedMessageByteArray
}

fun encryptMessage(message: String, password: String): String {
    val messageByteArray = message.encodeToByteArray()
    val passwordByteArray = password.encodeToByteArray()
    val encryptedMessageByteArray = encDecXorFun(messageByteArray, passwordByteArray)
    var encryptedMessageStringOfBits = ""

    for (i in encryptedMessageByteArray.indices) {
        encryptedMessageStringOfBits += (encryptedMessageByteArray[i].toInt().toString(2)).padStart(8, '0')
    }
    encryptedMessageStringOfBits += "000000000000000000000011"
    return encryptedMessageStringOfBits
}

fun hideTextInImage(image: BufferedImage, message: String): BufferedImage {
    val width = image.width
    val height = image.height
    val outputImage = BufferedImage(width, height, BufferedImage.TYPE_INT_RGB)
    var messageIndex = 0
    for (y in 0 until height) {
        for (x in 0 until width) {
            val color = Color(image.getRGB(x, y))
            val g = color.green
            val r = color.red
            var b = color.blue

            if (messageIndex < message.length) {
                b = changeLSB(color.blue, getCharacter(message, messageIndex))
                messageIndex++
            }
            val colorNew = Color(r, g, b)
            outputImage.setRGB(x, y, colorNew.rgb)  // Set the new color at the (x, y) position
        }
    }
    return outputImage
}

fun changeLSB(value: Int, bit: Int): Int {
    return if (bit == 1) {
        if (value % 2 == 0) value + 1 else value
    } else {
        if (value % 2 == 0) value else value - 1
    }
}

fun getCharacter(string: String, index: Int): Int {
    if (index >= string.length) {
        return 0
    }
    return Character.getNumericValue(string[index])
}

fun showMessageInImage() {
    println("Input image file:")
    val inputFileName = File(readLine()!!)
    println("Password:")
    val password = readLine()!!.encodeToByteArray()
    val image: BufferedImage = ImageIO.read(inputFileName)
    var messageInBits = ""
    outLoop@ for (y in 0 until image.height) {
        for (x in 0 until image.width) {
            val color = Color(image.getRGB(x, y))  // Read color from the (x, y) position
            messageInBits += getLSB(color.blue)
            if ("000000000000000000000011" in messageInBits && messageInBits.length % 8 == 0) {
                messageInBits = messageInBits.dropLast(24)
                break@outLoop
            }
        }
    }

    val messageByteArray = ByteArray(messageInBits.length / 8)

    for (i in messageInBits.indices step 8) {
        val stringByte = messageInBits.slice(i..i + 7)
        val byteValue = stringByte.toInt(2)
        messageByteArray[i / 8] = byteValue.toByte()
    }
    println()
    println("Message: \n${encDecXorFun(messageByteArray, password).toString(Charsets.UTF_8)}")
    println()
}

fun getLSB(colorValue: Int): Int {
    return colorValue % 2
}>
