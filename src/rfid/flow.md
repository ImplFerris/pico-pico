# Flow

<a href="./images/mifare-flow.png"><img style="display: block; margin: auto;" alt="MIFARE Memory layout" src="./images/mifare-flow.png"/></a>

When you bring a tag near the reader, the tag is powered by the reader RF field and is ready to respond to polling commands.

To check if any tag is nearby, we send a polling command. Normally this is REQA. If a tag was previously put into the HALT state, we use WUPA instead. If a tag is present and allowed to respond, it replies with ATQA (Answer To Request).

> [!Important]
> Note: Once the card is in the HALT state, only the WUPA (Wake Up) command can wake it up and let us do more operations. It always reminds me of Chandler from Friends saying "WOOPAAH". REQA works only on idle cards.

After receiving ATQA, we run the anticollision and select procedure. During this step, we handle collisions if multiple tags are present and retrieve the UID of one tag. Once this procedure completes, the card is selected and active.

After the card is selected, we authenticate the specific sector we want to read from or write to. Authentication must be performed again when accessing a different sector.

Once authentication succeeds, we can perform operations such as read, write, increment, decrement, or restore.

When all operations are done, we send the HLTA command to put the card into the HALT state. While the RF field remains on, a halted card will not respond to REQA and can only be reactivated using WUPA. If the card leaves the RF field and comes back, it starts again from the beginning.
