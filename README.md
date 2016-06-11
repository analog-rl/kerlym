# KEras Reinforcement Learning gYM agents, KeRLym

This repo is intended to host a handful of reinforcement learning agents implemented using the Keras (http://keras.io/) deep learning library for Theano and Tensorflow.
It is intended to make it easy to run, measure, and experiment with different learning configuration and underlying value function approximation networks while running a variery of OpenAI Gym environments (https://gym.openai.com/).

![alt tag](https://pbs.twimg.com/media/Ciq7nJJUYAEB4rK.jpg:large)


# Agents

 - pg: policy gradient method with Keras NN policy network
 - ddqn: double q-learning agent with various Keras NN's for Q approximation
 - dqn: q-learning agent with various Keras NN's for Q approximation

# Installation

```
sudo python setup.py install
```

# Usage

```
./run_pong.sh
```

or

```
Exmaple: kerlym -e Go9x9-v0 -n simple_dnn -P

Usage: kerlym [options]

Options:
  -h, --help            show this help message and exit
  -e ENV, --env=ENV     Which GYM Environment to run [MountainCar-v0]
  -n NET, --net=NET     Which NN Architecture to use for Q-Function
                        approximation [simple_dnn]
  -f UPDATE_FREQ, --update_freq=UPDATE_FREQ
                        Frequency of NN updates specified in time steps [1000]
  -u UPDATE_SIZE, --update_size=UPDATE_SIZE
                        Number of samples to train on each update [1100]
  -b BS, --batch_size=BS
                        Batch size durring NN training [32]
  -o DROPOUT, --dropout=DROPOUT
                        Dropout rate in Q-Fn NN [0.5]
  -p EPSILON, --epsilon=EPSILON
                        Exploration(1.0) vs Exploitation(0.0) action
                        probability [0.1]
  -D EPSILON_DECAY, --epsilon_decay=EPSILON_DECAY
                        Rate of epsilon decay: epsilon*=(1-decay) [0.0001]
  -s EPSILON_MIN, --epsilon_min=EPSILON_MIN
                        Min epsilon value after decay [0.05]
  -d DISCOUNT, --discount=DISCOUNT
                        Discount rate for future reards [0.99]
  -t NFRAMES, --num_frames=NFRAMES
                        Number of Sequential observations/timesteps to store
                        in a single example [2]
  -m MAXMEM, --max_mem=MAXMEM
                        Max number of samples to remember [100000]
  -P, --plots           Plot learning statistics while running [False]
  -F PLOT_RATE, --plot_rate=PLOT_RATE
                        Plot update rate in episodes [10]
  -S, --submit          Submit Results to OpenAI [False]
  -a AGENT, --agent=AGENT
                        Which learning algorithm to use [ddqn]
  -i, --difference      Compute Difference Image for Training [False]
  -r LEARNING_RATE, --learning_rate=LEARNING_RATE
                        RMSprop Learning Rate [0.0001]
```

or

```python
from gym import envs
env = envs.make("SpaceInvaders-v0")

import kerlym
agent = kerlym.agents.D2QN(env, 
                    nframes=1, 
                    epsilon=0.5, 
                    discount=0.99, 
                    modelfactory=kerlym.networks.simple_cnn,
                    update_nsamp=1000, 
                    batch_size=32, 
                    dropout=0.1, 
                    enable_plots = True, 
                    max_memory = 1000000, 
                    epsilon_schedule=lambda episode,epsilon: max(0.1, epsilon*(1-1e-4)),                
                    dufference_obs = True,
                    preprocessor = kerlym.perproc.karpathy_preproc,
                    learning_rate = 1e-3
                    )
agent.learn()
```

# Custom Action-Value Function Network

```
def custom_Q_nn(agent, env, dropout=0, h0_width=8, h1_width=8, **args):
    S = Input(shape=[agent.input_dim])
    h = Reshape([agent.nframes, agent.input_dim/agent.nframes])(S)
    h = TimeDistributed(Dense(h0_width, activation='relu', init='he_normal'))(h)
    h = Dropout(dropout)(h)
    h = LSTM(h1_width, return_sequences=True)(h)
    h = Dropout(dropout)(h)
    h = LSTM(h1_width)(h)
    h = Dropout(dropout)(h)
    V = Dense(env.action_space.n, activation='linear',init='zero')(h)
    model = Model(S,V)
    model.compile(loss='mse', optimizer=RMSprop(lr=0.01) )
    return model

agent = keras.agents.D2QN(env, modelfactory=custom_Q_nn)

```

# Citation

If using this work in your research, citation of our publication introducing this platform would be greatly appreciated!
The arXiv paper is available at https://arxiv.org/abs/1605.09221 and a simple bibtex entry is provided below.

```
@misc{1605.09221,
Author = {Timothy J. O'Shea and T. Charles Clancy},
Title = {Deep Reinforcement Learning Radio Control and Signal Detection with KeRLym, a Gym RL Agent},
Year = {2016},
Eprint = {arXiv:1605.09221},
}
```

# Acknowledgements

Much thanks to all of the following projects for their inspiration and contributions
 - https://github.com/dandxy89/rf_helicopter
 - https://github.com/sherjilozair/dqn
 - https://gist.github.com/karpathy/a4166c7fe253700972fcbc77e4ea32c5
 - Keras and Gym

-Tim

# Remind
Install `pip install gym` and `pip install gym[atari]`. If gym[atari] has install error, `apt-get install cmake`.
