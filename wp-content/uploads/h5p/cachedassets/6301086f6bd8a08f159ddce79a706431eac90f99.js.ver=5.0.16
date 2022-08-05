H5P.MemoryGame = (function (EventDispatcher, $) {

  /**
   * Memory Game Constructor
   *
   * @class
   * @param {Object} parameters
   * @param {Number} id
   */
  function MemoryGame(parameters, id) {
    var self = this;

    // Initialize event inheritance
    EventDispatcher.call(self);

    var flipped, timer, counter, popup, $feedback;
    var cards = [];
    var removed = 0;

    /**
     * Check if these two cards belongs together.
     *
     * @private
     * @param {H5P.MemoryGame.Card} card
     * @param {H5P.MemoryGame.Card} mate
     * @param {H5P.MemoryGame.Card} correct
     */
    var check = function (card, mate, correct) {
      if (mate === correct) {
        // Remove them from the game.
        card.remove();
        mate.remove();

        removed += 2;

        var finished = (removed === cards.length);
        var desc = card.getDescription();

        if (finished) {
           self.triggerXAPIScored(1, 1, 'completed');
        }

        if (desc !== undefined) {
          // Pause timer and show desciption.
          timer.stop();
          popup.show(desc, card.getImage(), function () {
            if (finished) {
              // Game has finished
              $feedback.addClass('h5p-show');
            }
            else {
              // Popup is closed, continue.
              timer.start();
            }
          });
        }
        else if (finished) {
          // Game has finished
          timer.stop();
          $feedback.addClass('h5p-show');
        }
      }
      else {
        // Flip them back
        card.flipBack();
        mate.flipBack();
      }
    };

    /**
     * Adds card to card list and set up a flip listener.
     *
     * @private
     * @param {H5P.MemoryGame.Card} card
     * @param {H5P.MemoryGame.Card} mate
     */
    var addCard = function (card, mate) {
      card.on('flip', function () {
        self.triggerXAPI('interacted');
        // Keep track of time spent
        timer.start();

        if (flipped !== undefined) {
          var matie = flipped;
          // Reset the flipped card.
          flipped = undefined;

          setTimeout(function () {
            check(card, matie, mate);
          }, 800);
        }
        else {
          // Keep track of the flipped card.
          flipped = card;
        }

        // Count number of cards turned
        counter.increment();
      });

      cards.push(card);
    };

    // Initialize cards.
    for (var i = 0; i < parameters.cards.length; i++) {
      if (MemoryGame.Card.isValid(parameters.cards[i])) {
        // Add two of each card
        var cardOne = new MemoryGame.Card(parameters.cards[i], id);
        var cardTwo = new MemoryGame.Card(parameters.cards[i], id);
        addCard(cardOne, cardTwo);
        addCard(cardTwo, cardOne);
      }
    }
    H5P.shuffleArray(cards);

    /**
     * Attach this game's html to the given container.
     *
     * @param {H5P.jQuery} $container
     */
    self.attach = function ($container) {
      this.triggerXAPI('attempted');
      // TODO: Only create on first!
      $container.addClass('h5p-memory-game').html('');

      // Add cards to list
      var $list = $('<ul/>');
      for (var i = 0; i < cards.length; i++) {
        cards[i].appendTo($list);
      }

      if ($list.children().length) {
        $list.appendTo($container);

        $feedback = $('<div class="h5p-feedback">' + parameters.l10n.feedback + '</div>').appendTo($container);

        // Add status bar
        var $status = $('<dl class="h5p-status">' +
                        '<dt>' + parameters.l10n.timeSpent + '</dt>' +
                        '<dd class="h5p-time-spent">0:00</dd>' +
                        '<dt>' + parameters.l10n.cardTurns + '</dt>' +
                        '<dd class="h5p-card-turns">0</dd>' +
                        '</dl>').appendTo($container);

        timer = new MemoryGame.Timer($status.find('.h5p-time-spent'));
        counter = new MemoryGame.Counter($status.find('.h5p-card-turns'));
        popup = new MemoryGame.Popup($container);

        $container.click(function () {
          popup.close();
        });
      }
    };
  }

  // Extends the event dispatcher
  MemoryGame.prototype = Object.create(EventDispatcher.prototype);
  MemoryGame.prototype.constructor = MemoryGame;

  return MemoryGame;
})(H5P.EventDispatcher, H5P.jQuery);
;
(function (MemoryGame, EventDispatcher, $) {

  /**
   * Controls all the operations for each card.
   *
   * @class H5P.MemoryGame.Card
   * @param {Object} parameters
   * @param {Number} id
   */
  MemoryGame.Card = function (parameters, id) {
    var self = this;

    // Initialize event inheritance
    EventDispatcher.call(self);

    var path = H5P.getPath(parameters.image.path, id);
    var width, height, margin, $card;

    var a = 96;
    if (parameters.image.width !== undefined && parameters.image.height !== undefined) {
      if (parameters.image.width > parameters.image.height) {
        width = a;
        height = parameters.image.height * (width / parameters.image.width);
        margin = '' + ((a - height) / 2) + 'px 0 0 0';
      }
      else {
        height = a;
        width = parameters.image.width * (height / parameters.image.height);
        margin = '0 0 0 ' + ((a - width) / 2) + 'px';
      }
    }
    else {
      width = height = a;
    }

    /**
     * Flip card.
     */
    self.flip = function () {
      $card.addClass('h5p-flipped');
      self.trigger('flip');
    };

    /**
     * Flip card back.
     */
    self.flipBack = function () {
      $card.removeClass('h5p-flipped');
    };

    /**
     * Remove.
     */
    self.remove = function () {
      $card.addClass('h5p-matched');
    };

    /**
     * Get card description.
     *
     * @returns {string}
     */
    self.getDescription = function () {
      return parameters.description;
    };

    /**
     * Get image clone.
     *
     * @returns {H5P.jQuery}
     */
    self.getImage = function () {
      return $card.find('img').clone();
    };

    /**
     * Append card to the given container.
     *
     * @param {H5P.jQuery} $container
     */
    self.appendTo = function ($container) {
      // TODO: Translate alt attr
      $card = $('<li class="h5p-memory-card" role="button" tabindex="1">' +
                  '<div class="h5p-front"></div>' +
                  '<div class="h5p-back">' +
                    '<img src="' + path + '" alt="Memory Card" width="' + width + '" height="' + height + '"' + (margin === undefined ? '' : ' style="margin:' + margin + '"') + '/>' +
                  '</div>' +
                  '</li>')
        .appendTo($container)
        .children('.h5p-front')
          .click(function () {
            self.flip();
          })
          .end();
      };
  };

  // Extends the event dispatcher
  MemoryGame.Card.prototype = Object.create(EventDispatcher.prototype);
  MemoryGame.Card.prototype.constructor = MemoryGame.Card;

  /**
   * Check to see if the given object corresponds with the semantics for
   * a memory game card.
   *
   * @param {object} params
   * @returns {boolean}
   */
  MemoryGame.Card.isValid = function (params) {
    return (params !== undefined &&
            params.image !== undefined &&
            params.image.path !== undefined);
  };

})(H5P.MemoryGame, H5P.EventDispatcher, H5P.jQuery);
;
(function (MemoryGame) {

  /**
   * Keeps track of the time spent.
   *
   * @class H5P.MemoryGame.Timer
   * @param {H5P.jQuery} $container
   */
  MemoryGame.Timer = function ($container) {
    var self = this;
    var interval, started, totalTime = 0;

    /**
     * Make timer more readable for humans.
     * @private
     * @param {Number} seconds
     * @returns {String}
     */
    var humanizeTime = function (seconds) {
      var minutes = Math.floor(seconds / 60);
      var hours = Math.floor(minutes / 60);

      minutes = minutes % 60;
      seconds = Math.floor(seconds % 60);

      var time = '';

      if (hours !== 0) {
        time += hours + ':';

        if (minutes < 10) {
          time += '0';
        }
      }

      time += minutes + ':';

      if (seconds < 10) {
        time += '0';
      }

      time += seconds;

      return time;
    };

    /**
     * Update the timer element.
     *
     * @private
     * @param {boolean} last
     * @returns {number}
     */
    var update = function (last) {
      var currentTime = (new Date().getTime() - started);
      $container.text(humanizeTime(Math.floor((totalTime + currentTime) / 1000)));

      if (last === true) {
        // This is the last update, stop timing interval.
        clearTimeout(interval);
      }
      else {
        // Use setTimeout since setInterval isn't safe.
        interval = setTimeout(function () {
          update();
        }, 1000);
      }

      return currentTime;
    };

    /**
     * Starts the counter.
     */
    self.start = function () {
      if (started === undefined) {
        started = new Date();
        update();
      }
    };

    /**
     * Stops the counter.
     */
    self.stop = function () {
      if (started !== undefined) {
        totalTime += update(true);
        started = undefined;
      }
    };

  };
  
})(H5P.MemoryGame);
;
(function (MemoryGame) {

  /**
   * Keeps track of the number of cards that has been turned
   *
   * @class H5P.MemoryGame.Counter
   * @param {H5P.jQuery} $container
   */
  MemoryGame.Counter = function ($container) {
    var self = this;

    var current = 0;

    /**
     * Increment the counter.
     */
    self.increment = function () {
      current++;
      $container.text(current);
    };
  };

})(H5P.MemoryGame);
;
(function (MemoryGame, $) {

  /**
   * A dialog for reading the description of a card.
   *
   * @class H5P.MemoryGame.Popup
   * @param {H5P.jQuery} $container
   */
  MemoryGame.Popup = function ($container) {
    var self = this;

    var closed;

    var $popup = $('<div class="h5p-memory-pop"><div class="h5p-memory-image"></div><div class="h5p-memory-desc"></div></div>').appendTo($container);
    var $desc = $popup.find('.h5p-memory-desc');
    var $image = $popup.find('.h5p-memory-image');

    /**
     * Show the popup.
     *
     * @param {string} desc
     * @param {H5P.jQuery} $img
     * @param {function} done
     */
    self.show = function (desc, $img, done) {
      $desc.html(desc);
      $img.appendTo($image.html(''));
      $popup.show();
      closed = done;
    };

    /**
     * Close the popup.
     */
    self.close = function () {
      if (closed !== undefined) {
        $popup.hide();
        closed();
        closed = undefined;
      }
    };
  };

})(H5P.MemoryGame, H5P.jQuery);
;
