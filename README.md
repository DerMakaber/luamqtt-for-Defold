# luamqtt for Defold

The luamqtt client prepared for Defold.
Available at https://github.com/xHasKx/luamqtt

# Installation
You can use luamqtt in your own project by adding this project as a [Defold library dependency](http://www.defold.com/manuals/libraries/).
Open your game.project file and in the dependencies field under project add:

https://github.com/DerMakaber/luamqtt-for-Defold/archive/master.zip

Or point to the ZIP file of a [specific release](https://github.com/DerMakaber/luamqtt-for-Defold/releases).

# Usage

Create a client

    local mqtt = require("mqtt.init")

    local BROKER = "test.mosquitto.org"  -- Address of the Broker
    local PORT = 1883 -- Port of the Broker
    local CLIENT_ID = "luamqtt-defold-sample-identifier" -- Client identifier, this must be unique

    client = mqtt.client{ uri = BROKER .. ":" .. PORT, id = CLIENT_ID, clean = true, reconnect = 3, version = 5}

Assign MQTT client event handlers

	client:on{
		connect = function(connack)
			if connack.rc ~= 0 then
				print("connection to broker failed:", connack:reason_string(), connack)
				return
			end
			-- connection established, now subscribe to test topic and publish a message after
			assert(client:subscribe{ topic="topic/sample", qos=1, callback=function()
				assert(client:publish{ topic="topic/sample", payload = "luamqtt client connected"})
			end})
		end,

		message = function(msg)
			assert(client:acknowledge(msg))

			print(string.format('Client received: %s.', msg))
		end,
	}

Initialise the ioloop for client

	  mqtt.init_ioloop(client)

Subscribe to a topic.

      client:subscribe{ topic='topic'}

Unsubscribe from a topic

      self.client:unsubscribe{ topic='topic'}

Iterate the ioloop

      mqtt.iterate_ioloop()

Publish a message to a topic.

      client:publish{ topic='topic', payload = "Testmessage from Defold"}

Disconnect the client.

      client:disconnect()

# Example

    local mqtt = require("mqtt.init")

    local BROKER = "test.mosquitto.org"  -- Address of the Broker
    local PORT = 1883 -- Port of the Broker
    local CLIENT_ID = "luamqtt-defold-sample-identifier" -- Client identifier, this must be unique

    function init(self)
        msg.post(".", "acquire_input_focus")
        msg.post("@render:", "clear_color", { color = vmath.vector4(0.4, 0.5, 0.8, 1.0) })

        -- define a topic
        self.topic = "topic/sample"
        
        -- create MQTT client
        self.client = mqtt.client{ uri = BROKER .. ":" .. PORT, id = CLIENT_ID, clean = true, reconnect = 3, version = 5}
        
        -- assign MQTT client event handlers
        self.client:on{
            connect = function(connack)
                if connack.rc ~= 0 then
                    print("connection to broker failed:", connack:reason_string(), connack)
                    return
                end
                -- connection established, now subscribe to test topic and publish a message after
                assert(self.client:subscribe{ topic=self.topic, qos=1, callback=function()
                    assert(self.client:publish{ topic=self.topic, payload = "luamqtt client connected"})
                end})
            end,

            message = function(msg)
                assert(self.client:acknowledge(msg))

                print(string.format('Client received: %s.', msg))
                gui.set_text(gui.get_node("incoming_text"), string.format("Hey, %s just received: %s.", msg.topic, msg.payload))

            end,
        }
        
        -- initialise the ioloop for client
        mqtt.init_ioloop(self.client)
    end

    function final(self)
        msg.post(".", "release_input_focus")

        -- Unsubscribe, disconnect and destroy on finalizing
        self.client:unsubscribe{ topic=self.topic}
        self.client:disconnect()
    end

    function update(self, dt)
        
        -- iterate the ioloop
        mqtt.iterate_ioloop()
    end

    function on_input(self, action_id, action)
        if action.pressed and gui.pick_node(gui.get_node("button"), action.x, action.y) then
            -- Publish to the Broker
            self.client:publish{ topic=self.topic, payload = "Testmessage from Defold"}
        end
    end

## Functions

Two new functions have been added so that luamqtt can be run with Defold. 

### mqtt.init_ioloop(client)
Used to initialise the ioloop, called once after the setup is done and the client is ready to be run.

**PARAMETERS**
* `client` (variable) - The variable of the created mqtt.client

    mqtt.init_ioloop(client)

### mqtt.iterate_ioloop()
Used to iterate the ioloop. Use this in your update() function

    function update(self, dt)
        mqtt.iterate_ioloop()
    end

## Reference

For the full reference please refer to https://xhaskx.github.io/luamqtt/

# Notes

I stripped out the unneeded files and made some smaller adjustments to the code. The only connection supported is through luasocket. (copas and ngx are removed)