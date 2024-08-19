+++
layout = 'post'
title = 'Batterarch'
date = 2024-08-18T12:00:00+05:45
draft = false
+++

# Batterarch Project
- view stock prices/dashboard in the terminal
#+begin_src elisp
;; to move a line down first get current line
;; transpose the line by +1, that will move the line to next row
;; col variable has the current-column, and it is later transposed by 1
(defun move-line-down ()
  (interactive)
  (let ((col (current-column)))
    (save-excursion
      (forward-line)
      (transpose-lines 1))
    (forward-line)
    (move-to-column col)))

;; follow same code but the column to transpose will be -1
(defun move-line-up ()
  (interactive)
  (let ((col (current-column)))
    (save-excursion
      (forward-line)
      (transpose-lines -1))
    (forward-line -1)
    (move-to-column col)))

(global-set-key (kbd "C-S-j") 'move-line-down)
(global-set-key (kbd "C-S-k") 'move-line-up)
#+end_src
